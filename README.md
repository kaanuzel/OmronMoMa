# OmronMoMa
**Introduction **
In this project, we combine a 6-DoF manipulator (Omron TM5-700) with an autonomous mobile robot (Omron LD-90). A PLC program is running to communicate with both robots. 

**PLC Program Structure **
PLC program includes a ModbusTCP Client to communicate with ModbusTCP server of TM5-700, a TCP/IP Client to communicate with TCP/IP server of LD-90, and an OPCUA server to allow control access to external OPCUA clients.
This program is running on TwinCAT automation software in the PC, it is programmed with Structured Text (ST) language. 

ModbusTCP Client shown as TM5-700 Client section in figure 1 has two function blocks, FB_ ModbusWriteTM and FB_ ModbusReadTM, for writing data to and reading data from the ModbusTCP server respectively. These function blocks are responsible for communicating with the corresponding data addresses of the server in terms of word and byte array. They make use of the Tc2_ModbusSrv library of Beckhoff. Methods to read/write Joint/cartesian positions and working status of the manipulator are implemented in TM-500 functions section. This section has two function blocks called FB_ HandleSetTM and FB_ HandleGetTM. FB_ HandleSetTM function block includes methods such as MoveCartesian() and SetJoints(), and it calls the FB_ ModbusWriteTM function block to write the command on the server with the command ID of the method called. FB_ HandleGetTM function block has methods such as GetPositions() and IsError() to be read from the server by calling the FB_ ModbusReadTM function block.  

TCP/IP client shown as LD-90 Client section in Figure 2 has one function block called FB_ SocketLD. This function block starts the socket communication with the TCP/IP server and is responsible for both sending and receiving byte arrays and parsing the string commands. It makes use of Tc2_TcpIp library of Beckhoff. Methods to read/write commands to the mobile robot are implemented in LD-90 Functions section in the function block FB_ HandleRobotLD. This function blocks includes methods such as GetStatus() and GoToTarget(). For both get and set methods, FB_ SocketLD function block is called.  

The Mobile Manipulator 9-DoF section includes one function block called FB_MobileManipulator9DoF. This function block is calling the methods for both the manipulator and the mobile robot when requested. It has boolean variables for each method, where the methods are called with a rising edge of the booleans. These boolean variables are attributed to the OPCUA server to be published to the OPCUA clients. FB_MobileManipulator9DoF is also responsible for controlling the running order for both robots.  

OPCUA Server section represents the OPCUA Server Project that maps the server to the attributed PLC variables. In our case, these variables are the robot status and Booleans for method calls. The server publishes these variables with the authorized clients.   

**Communication of PLC with TM5-700 Manipulator ** 
TM5-700 manipulator is programmed using TMFlow software. This software alows the user to program the robot using blocks. It also provides a ModbusTCP server for external control. Robot parameters to be read and to be written have unique addresses in the provided ModbusTCP Code Table. In addition, new ModbusTCP parameters can be defined in the provided empty addresses. For this project, new modbus addresses such as command ID and positions are defined in TMFlow program, that are listed in the excel sheet. TMFlow program called ModbusHello uses these variables to execute Move and other blocks. These Modbus variables are prepared by the methods called in the PLC and written into the corresponding Modbus addresses using FB_ ModbusWriteTM function block.  

**Communication of PLC with LD-90 Mobile Robot **
LD-90 mobile robot is programmed using MobilePlanner software and ARCL (Advanced robot Command Language) commands. ARCL commands can be used from the command prompt by calling Telnet 1.2.3.4 7171. When accessed externally, the TCP/IP server of the robot provides string responses to the clients as byte arrays, and it also receives string commands as byte arrays. Function blocks FB_ HandleRobotLD and FB_SocketLD connect to this server and enter the password (admin) automatically to allow mobile robot methods to be called.  

**Communication of OPCUA Clients with PLC **
IP address of the OPCUA server is opc.tcp://169.254.220.9:4840 . The server uses Basic256Sha256 Encryption method for data transfer, and it requires sign in for connection. Username/password are admin/admin. 
