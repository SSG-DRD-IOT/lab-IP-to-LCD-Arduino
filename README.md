# Introduction to I<sup>2</sup>C and the LCD Display

## Objective

The objective of this lab is a learn the basics of an example that get the IP address of the current Linux based IoT device and displays it on the LCD.

## Hardware requirements

Module | Pin
--- | ---
LCD Display | Any I<sup>2</sup>C Port

![](./images/action.png) Connect **LCD Display** to Any I<sup>2</sup>C Port.

## I<sup>2</sup>C using the MRAA API
Create a new project
```c
#include <stdio.h>
#include <stdlib.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netdb.h>
#include <ifaddrs.h>

#include "jhd1313m1.h"
#include "upm.h"
#include "upm_utilities.h"
#include "signal.h"
#include "string.h"

char str1[1];

char *get_ip_address(){
  char *addr;
  struct ifaddrs *ifaddr, *ifa;
  struct sockaddr_in *sa;
  
  if (getifaddrs( & ifaddr) == -1) {
    perror("getifaddrs");
    exit(EXIT_FAILURE);
  }

  for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
    if (ifa->ifa_addr == NULL)
      continue;

    if (ifa->ifa_addr->sa_family == AF_INET) {
      str1[0] = ifa->ifa_name[0];
      if (strcmp(str1, "e") == 0) {

        sa = (struct sockaddr_in * ) ifa->ifa_addr;
        addr = inet_ntoa(sa->sin_addr);
        printf("Interface: %s\tAddress: %s\n", ifa->ifa_name, addr);

      };
    }
  }

  freeifaddrs(ifaddr);
  return addr;
}

int main() {

  // Set the subplatform
  mraa_add_subplatform(MRAA_GROVEPI, "0");
  
  // Get the IP address of the Up2 Board
  char *addr = get_ip_address();


  // initialize a JHD1313m1 on I2C bus 0, LCD address 0x3e, RGB
  // address 0x62
  jhd1313m1_context lcd = jhd1313m1_init(0, 0x3e, 0x62);

  if (!lcd) {
    printf("jhd1313m1_i2c_init() failed\n");
    return 1;
  }

  char str2[20];

  snprintf(str2, sizeof(str2), "%s", addr);

  jhd1313m1_set_cursor(lcd, 0, 2);

  jhd1313m1_write(lcd, "IP Address", strlen("IP Address"));

  jhd1313m1_set_cursor(lcd, 1, 2);

  jhd1313m1_write(lcd, str2, strlen(str2));

  // Change the color to Intel Blue ;)
  uint8_t r = 0;
  uint8_t g = 113;
  uint8_t b = 197;

  jhd1313m1_set_color(lcd, r, g, b);

  upm_delay(1);


  return 0;
}
```
## Using the Arduino API
```c++
#include "jhd1313m1.hpp"
#include "Arduino.h"

// initialize a JHD1313m1 on I2C bus 0, LCD address 0x3e, RGB
// address 0x62

void setup() {

  // Set the subplatform
  mraa_add_subplatform(MRAA_GROVEPI, "0");

  upm::Jhd1313m1 lcd(0, 0x3e, 0x62);

  // Get the IP address of the Up2 Board
  String addr = System.runShellCommand("hostname -I | cut -f1 -d \" \"");

  CloudSerial.begin(115200);
  CloudSerial.println(addr);

  lcd.setCursor(0, 2);

  lcd.write("IP Address");

  lcd.setCursor(1, 2);

  lcd.write(addr.c_str());

  // Change the color to Intel Blue ;)
  uint8_t r = 0;
  uint8_t g = 113;
  uint8_t b = 197;

  lcd.setColor(r, g, b);
}

void loop() {
  delay(1000);
}
```


## Additional resources
Information, community forums, articles, tutorials and more can be found at the [Intel Developer Zone](https://software.intel.com/iot).

For reference code for any sensor/actuator from the Grove* IoT Commercial Developer Kit, visit [https://software.intel.com/en-us/iot/hardware/sensors](https://software.intel.com/en-us/iot/hardware/sensors)
