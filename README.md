DJI Guidance SDK
========================

Official Guidance SDK package for accessing the rich categories of output data from Guidance via USB and UART.

Document
============

English version:


　[Developer Guide](doc/DeveloperGuide/DeveloperGuide.md)

　[Guidance SDK API Documentation](doc/Guidance_SDK_API.md)


中文版本：

　[开发者指南](doc/开发者指南/开发者指南.md)　

　[Guidance SDK API Documentation](doc/Guidance_SDK_API.md)


한글버전：

　[개발자 가이드](doc/개발자가이드/개발자가이드.md)　

　[Guidance SDK API 문서](doc/Guidance_SDK_API_kr.md)


Structure
=========
-	**demo**: a visual tracking project by using Guidance SDK
-	**doc**: API details
-	**examples**: examples for USB, UART and ROS
-	**include**: Header file of Guidance SDK 
-	**lib**: Library files for Windows
-	**so**: Library files for Linux

Also notice that, to enable fast download for ROS users, we have a separate ROS repo with much smaller size: [GuidanceSDKROS](https://github.com/dji-sdk/GuidanceSDKROS).

Usage
=========
### Windows ###

Examples of USB and UART can be found in *examples/usb\_example*, *examples/uart\_example*,	including Visual Studio projects which is ready to compile.  

### Linux ###

Examples of USB and UART can be found in *examples/usb\_example*, *examples/uart\_example*,	including Makefile which is ready to compile. 

A **ROS** package is also included in *examples/RosNode*.


