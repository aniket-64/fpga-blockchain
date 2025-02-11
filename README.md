# fpga-blockchain  

The purpose of this project is to implement a Key-Value store on FPGA
using Cuckoo Hashing. A Key-Value pair(KVP) is a fundamental data representation of two associated data items: a key and a value, where the key acts as
a unique identifier for the values. After KVPs are inserted into the system, they
can searched or deleted with the corresponding key only. We have used Cuckoo
hashing to implement the system using Verilog as our programming language to
program the FPGA.   

### Assignment-1
  
This assignment was done with the goal to get familiar with the Verilog HDL. One bit comparator, one bit adder, eight bit comparator and eight bit adder were
implemented.  

### Assignment-2
  
This assignment involved the implementation of Cuckoo hashing using Verilog, and its testing using a testbench.
Cuckoo Hashing is a form of open addressing in which each non-empty cell of
a hash table contains a key-value pair.A hash function is used to determine
location for each key and its presence in the table can be determined by examining the cell of that table.It is a technique for resolving collisions in hash tables
that produces a dictionary with constant-time worst-case lookup and deletion
operations as well as amortized constant-time insertion operations.
  
  
### Assignment-3
  
Assignment 3 involved implementation of UART transmitter and reciever modules. These modules were tested using a testbench. UART stands for Universal Asynchronous Receiver-Transmitter, which is a
computer hardware for asynchronous serial communication in which the transmission speeds are configurable. It is based on serial data transfer, which involves bit by bit transfer of the data bits, from the least significant bit to
the most significant bit. UART is an asynchronous communication protocol,
viz. There is no clock synchronisation between the receiving and transmitting
UARTs. Thus both the receiving and transmitting systems must agree on a predecided but configurable frequency, called the ‘Baud rate’.The UART makes use
of Start bit and Stop bit to handle the timing of data transmission.  

Transmitter Module:
This module receives the data in the form of an 8 bit array and the start, stop
and parity bit and then transfers it to the receiver module.  

Receiver Module:
This module receives the data from the Transmitter module, samples the data
to only store the middle of every tick and then remove the start, stop and parity
bits and the transfers the data bus to the receiving end.

### FPGA-Keyvalue Assignment

This is the final assignment of the project where we actually connect all the dots and actually implement the main aim of our project.  

##### Block RAM
Block RAMs (or BRAMs) stands for Block Random Access Memory. Block
RAMs are used for storing large amounts of data inside of the FPGA. We have used BRAM to store all the tables corresponding to the key-value database.
We have used Single Port Block
RAM configuration to read data from .txt files and store them in arrays. These .txt files are used to initialise the key-value database.
The .txt files used are:   

- value_valueadd.txt: This file contains the values to the corresponding
value addresses.  

- hashtable1_key.txt: This file relates a particular key to a particular
index which is the output of the first hash function when the said key is
its input. We initialized some indices with their corresponding key values
and the rest are initialized to 0.  

- hashtable1_valadd.txt: This file contains the value addresses of corresponding keys in hashtable1 key.txt.  

- hashtable2_key.txt: This file relates a particular key to a particular
index which is the output of the second hash function when the said key is
its input. We initialized some indices with their corresponding key values
and the rest are initialized to 0.  

- hashtable2_valadd.txt: This file contains the value addresses of corresponding keys in hashtable2 key.txt.
  
  
NOTE: Since 0 is used to indicate that a given index is not initialised, 0 cannot
be used as a key. Similarly, the value address 0 is not alloted to any value, as it
is used to indicate an uninitialised index in the key to value address mapping
tables.  

##### Operations on KVS
Then these arrays are used for the main Key Value Store/Database operations : INSERT, SEARCH
and TRANSACT:  
- INSERT: This operation is used to insert a key-value pair (using analogy of a bank account,
the key can be thought of the account number and the value is the money present
in the account) into the hash tables, using Cuckoo Hashing algorithm.  

- SEARCH: We use this operation to search for a particular key and then display the corresponding value against the key. Again, we use Cuckoo Hashing algorithm to
implement the search which reduces the time complexity to O(1)(worst case).
The Key goes through the Hashing functions which gives us 2 possible indices
where the key might be stored at. We then check both these locations for the
Key. If it is present in either the locations then we return the value corresponding to the key.  
  
- TRANSACT: This operation either debits or credits amount from the value of the given key. The input transact_kind is used to tell whether the amount is to be debited or credited. 0 stands for debit, 1 for credit.   
  
NOTE: The INSERT operation has been so designed that it imposes the restriction that the value for any key must be non zero.  

The file create_bram.v contains code for initialising the BRAM and implementations of these operations on the key-value stores.
It has been tested using the test bench file - create_bram_tb.v.  

##### Data Packet  

The data packet is of the following form:

- for refer:
command-code 1 byte  
key 4 bytes  
trailing zero 1 byte (00000000) used for detection purposes.  

- for transfer:
command-code 1 byte  
key-debit 4 bytes  
key-credit 4 bytes  
transfer amnt  
trailing zero 1 byte  


- for issue:
command-code 1 byte  
key 4 bytes  
amount 4 bytes  
trailing zero 1 byte  

- for create:
command-code 1 byte  
key 4 bytes  
initial balance 4 bytes  
trailing zero 1 byte  
  
##### Core Modules and Command Extractor
The main purpose of the Core Modules - refer.v; transfer.v; issue.v; create.v is to store the data received by
the receiver module, and use the data for executing the
4 functions - REFER, TRANSFER, ISSUE and CREATE.  

- create.v: The data is received by the receiver module of UDP/UART. Each time a new byte of
data is received, the receiver module negates the value of newbyt. The always
block always@(newbyt) is hence triggered, which stores this newly received byte
of data into an array [7:0]packet[11:0]. Once all the required number of bytes
are stored into this array, a trigger sets off, which leads to the execution of
another always block. It is responsible for updating the output values of the
output ports- signal, key and value. The main extractor module in cmdextract.v
detects a change in the outputs of the create module through an always block.
This always block then updates the values of the inputs of the create bram
module, such that the INSERT functionality detailed in the previous chapter is
triggered. This results in the creation and insertion of a new key-value pair into
the hash tables.  

- refer.v: Similar to the create module, the incoming data is stored byte-wise in the array.
Once all bytes of the packet are received, a block is triggered which changes
the outputs- signal and key, appropriately. The extractor module detects these
changes and directs these values to the create bram module. The SEARCH
functionality is triggered and the value corresponding to the key is returned.
  
 - issue.v: Similar to the create module, the incoming data is stored byte-wise in the array.
Once all bytes of the packet are received, a block is triggered which changes the
outputs- signal, key, transact value, and transact kind appropriately. The extractor module detects these changes and directs these values to the create bram
module. The TRANSACT functionality is triggered and the required amount
is issued to the key.  

- transfer.v: This module invokes the TRANSACT functionality of our main code, twice. To
take input from the UDP we follow a similar algorithm, getting Keys participating in the transaction. The the outputs are updated sequentially, first to debit
the amount from the sender, and then credit it to the receiver. The extractor module detects these changes and directs these values to the create bram
module Thus, the transaction takes place.  

These modules are controlled by the command extractor module - cmdextract.v, which directs the data to the correct core module. 
This has been tested using the test bench cmd_tb.v.
