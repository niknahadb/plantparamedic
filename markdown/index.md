---
title: 'The Plant Paramedic: An Automated Watering System'

toc-title: 'Table of Contents'
abstract-title: '<h2>Description</h2>'
abstract: 'Our product, named the Plant Paramedic, is an automated plant irrigation system that ensures soil moisture levels are maintained at optimal levels for ideal plant health. Our product also allows the user to manually water their plants and receive an email with the moisture level and a timestamp every time the system stops watering. The soil moisture sensor reads the values from the soil, which are then sent to the CC3200 using GPIO input. These values are compared against the threshold values in our code to determine whether the soil is dry or wet, and that dictates the motor turning on and off. We also set up the IR receiver for manual control via the AT\&T remote. Finally, AWS IoT notifies the user via email about the current status of the system. All of these features combined provide the user with an intuitive automated product that is simple, smooth, and reliable.

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;padding-top:20px">
  <div style="display: inline-block; vertical-align: bottom;">
    <img src="./media/plant.png" style="width:5in;height:6in;"/>
    <!-- <span class="caption"> </span> -->
  </div>
</div>

<h2>Video Demo</h2>
<div style="text-align:center;margin:auto;max-width:560px">
  <div style="padding-bottom:56.25%;position:relative;height:0;">
    <iframe style="left:0;top:0;width:100%;height:100%;position:absolute;" width="560" height="315" src="https://www.youtube.com/embed/9ow2u0GxX7k" title="YouTube video player" frameborder="0" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
</div>

</div>'
---

<!-- EDIT METADATA ABOVE FOR CONTENTS TO APPEAR ABOVE THE TABLE OF CONTENTS -->
<!-- ALL CONTENT THAT FOLLWOWS WILL APPEAR IN AND AFTER THE TABLE OF CONTENTS -->

# Design

## Functional Specification

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 300px;">
The first state of our product, as seen in the State Diagram, is the monitoring or idle state, wherein the soil moisture sensor continually reads values while the IR receiver checks if the "LAST" button on the remote is pressed. Next, when the "LAST" button is pressed, the state changes to the watering state. In this state, the driver board sets the GPIO outputs to high for the motor to pump water. When the "MUTE" button is pressed, the system returns to the monitoring/idle state. If moisture levels are between 0.85 and 1.1, the motor turns on again, and water is pumped into the soil. Once the moisture levels drop below 0.85, the motor stops, and the system returns to the monitoring state.

  </div>
  <div style="display:inline-block;vertical-align:top;flex:0 0 500px">
    <div class="fig">
      <img src="./media/state.png" style="width:85%;height:auto;" />
      <span class="caption">State Diagram</span>
    </div>
  </div>
</div>

## System Architecture

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 400px;">
In our design, the two CC3200s served as bases. One was a monitoring base, and the other was a display. The first launchpad had the soil moisture sensor, IR receiver, DC motor, and water pump attached to it. It also connected to AWS through Wi-Fi to send emails to the user. This board also contained the voltage divider for the soil moisture sensor. The second CC3200 was linked to the monitoring one through UART and displayed the humidity levels on its OLED screen via SPI.

  </div>
  <div style="display:inline-block;vertical-align:top;flex:0 0 400px;">
    <div class="fig">
      <img src="./media/block.png" style="width:90%;height:auto;" />
      <span class="caption">Block Diagram</span>
    </div>
  </div>
</div>

# Implementation

The first piece of hardware we implemented was the soil moisture sensor. This was plugged into a comparator which had Vin, GND, and A0. Vin was connected to VCC, and GND to GND. The A0 pin was wired to pin 60 on our CC3200, which was configured as channel 3 of ADC input. Between the A0 pin and pin 60, we built a voltage divider circuit using two 3.3k Ohm resistors because the input voltage exceeded the maximum of 1.4V that the CC3200 is rated for. The output of the voltage divider circuit was fed into pin 60, completing the hardware setup for the soil moisture sensor. For software, we used ADC demo from the TI examples folder to help build the code for this component. ADC demo contains the code on how to set up the sensor and manipulate output to your desire. We read in raw ADC values, multiplied it by 1.4, and then divided that number by 4096 to obtain the humidity level of the soil in voltages.

The DC motor with the driver board was the next hardware component to be set up. The red and black wires of the motor were inserted into out1 and out2 of the driver board (alternatively, you could use out3 and out4), and the driver board was powered by the 5V output from the CC3200. The enable, in1, and in2 were then connected to pins 5, 63, and 64, respectively. These could be configured to any GPIO output pins using the SysConfig tool. In order for the motor to work, we used the PWM demo file from the TI examples folder and edited the code to remove irrelevant functions, retaining only one function that sets enable to high to turn the motor on and turns it off when enable is low. Setting up the water pump motor attached to the driver board is very simple. All you have to do is attach the tube that is included in the package to the motor, and that is it. The motor is obviously water proof, so you could drop it in a cup of water and it will push the water through the tube.

The IR receiver was set up similarly to Lab 3. GND and VCC were connected to their respective pins, and the output was connected to pin 50 on the CC3200 as GPIO input. The code for this was exactly the same as Lab 3, which included matching the binary values received from the IR sensor with the corresponding character being pressed on the AT\&T remote. This code also used SysTick and interrupts extensively to achieve this. To implement interrupts, you have to set up and register your interrupt handlers, then create a function called GPIOIntHandler() that uses binary shifting and interrupts to decode the incoming IR signals.

The OLED board was connected using SPI to the launchpad, and corresponding pins were connected accordingly. MOSI, CLK, DC, CS, and RESET were the only pins we needed to connect (excluding VCC and GND). The code to set up the OLED board included setting up SPI Communication, as well as making sure the correct pins are assigned to the write and read commands in Adafruit_OLED.c. To enable UART communication across both CC3200 launchpads, we first set up UART communication using the code from previous labs on registering and enabling.connected the first CC3200’s RX to the second CC3200’s TX, and its TX to the second’s RX. We also connected a common ground between the two launchpads.

The next step is to set up the CC3200 to connect AWS Servers. If you already haven't done so in lab 4, you need to create an Amazon AWS account to access the AWS IoT Console. Create a Thing, representing your physical device, and save the REST API endpoint address. Generate and download the required certificates: public key, private key, RootCA certificate, and any other certificates, and ensure they are activated. Set up a device shadow to represent the CC3200 in the cloud, and configure the necessary permissions. Create a new policy in Secure Policies and attach it to the certificate. Convert your private key and certificates from .pem to .der format using openSSL, and update your CC3200 to the latest service pack with UniFlash. Flash the .der certificates and private key to the board using UniFlash. Configure the SimpleLink socket library with the correct socket credentials and connect the board to the internet in common.h. Update the date and time variables in main.c to validate the credentials.

To send an email, you need to essentially send it as a POST request. Please refer to the main.c in aws-rest-api-ssl-demo for the function on how to send a POST request called http_post(). This function initializes buffers for the request and server response, sets up headers and content length, and appends the actual data to be sent in the POST body. The constructed POST request is sent over the TLS socket using sl_Send(), with error handling for the send and receive operations. The function then prints the server's response to UART for debugging and returns 0 upon successful execution.

Please refer to our circuit schematic below for all the components, wire connections, and pins.

  <div style='display: inline-block; vertical-align: top;flex:1 0 600px'>
    <div class="fig">
      <img src="./media/schem.png" style="width:auto;height:auto;padding-top:30px" />
      <span class="caption">Circuit Schematic</span>
    </div>
  </div>

# Challenges

The initial challenge we faced during our implementation was that the CC3200 was supplying too much input voltage to our soil moisture sensor, causing it to consistently read the maximum voltage of 1.4 V instead of the actual moisture level. To solve this problem, we constructed a voltage divider on our board to reduce the input voltage from the CC3200. By doing so, we were finally able to obtain accurate readings from our soil moisture sensor.

Another challenge we encountered was our DC motor driver board inconsistently powering on and off. Debugging this issue was more complex, as it was a hardware-related problem. When we switched to a different driver board, it only worked as intended for a few minutes before reverting to the inconsistent behavior we had been troubleshooting earlier. We were able to narrow down the issue and discovered that when the water pump turned on, one of the input wires connected to the pump would become dislodged. When the wire disconnected even for half a second, it triggered an input/output error in our program, halting the entire process. To prevent the wire from loosening so often, we stripped it a bit more and fully tightened the screws on the driver board to hold it down. However, the initial startup of the water pump still caused the wire to move slightly. We finally resolved this issue by securing the wire so that it would not move at all when the pump powered on. After this, we no longer experienced any input/output errors or program stalls.

The third obstacle we faced was an unresponsive IR receiver sensor. This issue took longer to resolve as we initially believed it was a software-related problem. However, after replacing the old IR receiver with a new one, it worked as expected.

There were no other significant challenges or obstacles throughout the design and implementation process. The majority of the code we used from previous labs worked as expected, and there were no issues with connecting and communicating to AWS.

# Future Work

To further improve the design and user interface (UI) of our project, we would have liked to add a more interactive UI to our OLED by including more content, user control, and animations. This would include a welcome screen, a main menu to select between manual and automatic control, as well as a watering animation every time the plant is being watered. Currently, the OLED simply displays the live reading of the soil humidity level, so there is very little interaction between the user and our OLED display. Also during the demo, one of the lab members had to position the tube and water pump towards the plant since it was taller than where our boards were placed on the table. So we would also like to attach a frame for the tube and motor to rest on, eliminating the need for someone to hold the tube upright, thereby improving the overall efficiency of the system.

# Bill of Materials

<ol>
    <li>
        CC3200 Launchpad
        <br>
        Price: $66.00
        <br>
        Quantity: 2
        <br>
        Provided during lab (<a href="https://www.digikey.com/en/products/detail/texas-instruments/CC3200-LAUNCHXL/4862812?utm_adgroup=&utm_source=google&utm_medium=cpc&utm_campaign=PMax%20Shopping_Product_Low%20ROAS%20Categories&utm_term=&utm_content=&utm_id=go_cmp-20243063506_adg-_ad-__dev-c_ext-_prd-4862812_sig-CjwKCAjwr7ayBhAPEiwA6EIGxEtMye_Fat0JvejKsG5a4TxCuTJ-bT_BPEgVFcegrO3TbyRbO_UWARoCLNoQAvD_BwE&gad_source=4&gclid=CjwKCAjwr7ayBhAPEiwA6EIGxEtMye_Fat0JvejKsG5a4TxCuTJ-bT_BPEgVFcegrO3TbyRbO_UWARoCLNoQAvD_BwE">link</a>)
    </li>
    <br>
    <li>
        Adafruit OLED Display
        <br>
        Price: $38.95
        <br>
        Quantity: 1
        <br>
        Provided during lab (<a href="https://www.amazon.com/Adafruit-OLED-Breakout-Board-microSD/dp/B00XW2ME84">link</a>)
    </li>
    <br>
    <li>
        IR Receiver
        <br>
        Price: $1.95
        <br>
        Quantity: 1
        <br>
        Provided during lab (<a href="https://www.digikey.com/en/products/detail/adafruit-industries-llc/157/17039174?utm_adgroup=&utm_source=google&utm_medium=cpc&utm_campaign=Pmax_Shopping_Product_Silicon%20Valley%20Category%20Awareness&utm_term=&utm_content=&utm_id=go_cmp-20773039395_adg-_ad-__dev-c_ext-_prd-17039174_sig-CjwKCAjwr7ayBhAPEiwA6EIGxF6ovFOFuNgT9b5q3V55zkqflwOIITkc-EusKrdMglU1R3i_zLNFYhoCT6YQAvD_BwE&gad_source=1&gclid=CjwKCAjwr7ayBhAPEiwA6EIGxF6ovFOFuNgT9b5q3V55zkqflwOIITkc-EusKrdMglU1R3i_zLNFYhoCT6YQAvD_BwE">link</a>)
    </li>
    <br>
    <li>
        Soil Moisture Sensor
        <br>
        Price: $7.89
        <br>
        Quantity: 1
        <br>
        Purchased from Amazon (<a href="https://www.amazon.com/HiLetgo-Moisture-Automatic-Watering-Arduino/dp/B01DKISKLO/ref=sr_1_4?crid=1MSFOJYEEZ88W&dib=eyJ2IjoiMSJ9.8bMNU5rD4f1KbFOcivM_v7trta6A883U71Gmd5HPFXMUHfDCpNBEqs96Y6NH0_f7rgauObgY9IqOKxLPsZK4ZAt-fHgHY2BvV7TNjS6VTIDU_mBjBz3OqQpWRrIPMJx63mO-Kdz_mKDwbBs72srllZNqFIrgyRwvmB-GNFxygp0rWQx2jdzYNxI-2XbQiFfUoBnr04tPIPMyIXk89Ivgm8sffD3x4aMEZFHMnKqd9_w.9-ORyYiM7HQB73K4q17hFoCVLgdpWTZrbiHyENucyY4&dib_tag=se&keywords=soil+humidity+sensor+i2c+ti+launchpad&qid=1716394630&sprefix=soil+humidity+sensor+i2c+ti+launchpad%2Caps%2C238&sr=8-4">link</a>)
    </li>
    <br>
    <li>
        Water Pump + Tube
        <br>
        Price: $9.99
        <br>
        Quantity: 1
        <br>
        Purchased from Amazon (<a href="https://www.amazon.com/Sipytoph-Submersible-Flexible-Aquariums-Hydroponics/dp/B097F4576N/ref=sr_1_5?crid=VGO5OH2LELUP&dib=eyJ2IjoiMSJ9.57MNaBmzvgLjLh8H7YYxaQgUc5vO7Ev_nXlYmn4WMGNCEjbJUUjvW9ZEdVK6YXKzS9bNKdmSi2C_g327DzUwUn0_MfMN5PH_MZ7zNqqxCXuwSLpTZpdnORICBcmpAIsPWt2UzZwGnmDCQULWoFYmLYnnBmEhnrR6rmH5U9cd9yHttFzC0_E6cWuLrwxg071XR9Q05OdkR7a_iVfcx67r7Ti3XMlsGKqqdHgBMBJ49547nl-hp53L5MxqTCaHOdQLAbUZqzyInJTVFMkif7Cqzl1oKgxIA-njc8iLU8q2Kgc.1BXTkb1B_hGHzC7-i5-Cy28G0yv7KZhTRNp9tUcSZ-0&dib_tag=se&keywords=arduino+water+pump&qid=1716340868&sprefix=arduino+water+pump%2Caps%2C219&sr=8-5">link</a>)
    </li>
    <br>
    <li>
        DC Motor Driver Board
        <br>
        Price: $9.99
        <br>
        Quantity: 1
        <br>
        Purchased from Amazon (<a href="https://www.amazon.com/dp/B0C5JCF5RS?psc=1&ref=ppx_yo2ov_dt_b_product_details">link</a>)
    </li>
    <br>
    <li>
        3.3k Ohm Resistor
        <br>
        Price: $5.99
        <br>
        Quantity: 2
        <br>
        Purchased from Amazon (<a href="https://www.amazon.com/EDGELEC-Resistor-Tolerance-Multiple-Resistance/dp/B07QK9TX9C/ref=sr_1_1_sspa?crid=QGHFEEY6YK16&dib=eyJ2IjoiMSJ9.Gg7jzKjWXaR_be4hkniUsFFZJyyCLGDy4wYAZgS9kb-wrJmt_q1wg1wV3xXlarkHrqxSLsQH47aGfd_5LdIVkLZ9SjpDbfFwf4Fp4HK9AUIpp58l_ExXpqIy8LmlfZdEZeSmwC8fuitPr0fIaNSiZyhaCjYlJJP-pS_pIyEhfu3PSDFACU73g4HBb13cc5mOxdIdTTQujLVHPsOxlgqsiU8YIPpBWRey6YSRCJUMaak.eCo2FTArlv7ZrToBtaXqKMIoLXA50H95jQ-Jud-fBJc&dib_tag=se&keywords=3.3k+ohm+resistor&qid=1717729757&sprefix=3.3k+%2Caps%2C161&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1">link</a>)
    </li>
</ol>
