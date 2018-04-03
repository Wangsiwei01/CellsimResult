# CellsimResult

This repo is to reproduce the result of Sprout (http://alfalfa.mit.edu/) and compare the performance with Cubic and Vegas using Cellsim.
The enviorment contains 3 VMs (Cellism, Server and Client.)
Cellsim has two adapters, eth0 should be connected with the server and eth1 should be connected with the client.
For detailed set up please follow the instructions from http://alfalfa.mit.edu/

Be aware that when I installed Sprout, some instructions from http://alfalfa.mit.edu/ seems to be out of date.
  For example: sudo apt-get install libboost-math1.50-dev libboost-math1.50.0 libprotobuf7 libprotobuf-dev 
  These package seems to be not available. The reason seems to be the package version is pretty old. I use the command sudo apt-get install libboost-all-dev and installed a different version(libboost-all-dev)

After setting up the enviorment, we can start testing. Here is anther work we can refer: https://reproducingnetworkresearch.wordpress.com/2014/06/03/cs244-14-sprout/

For Sprout: 
  Sprout has its own applicaiton.
  First we run the Sprout server: 
  
     ./sproutbt2 
  And for the Sprout client:
  
    ./sproutbt2 server_address 60001
    

For Vegas or Cubic:
  To install: 
  
    sudo modprobe -q tcp_vegas(or tcp_cubic) 
    sudo bash -c ‘echo vegas > /proc/sys/net/ipv4/tcp_congestion_control'
  Then the server: 
  
    iperf -s
  For the client: 
  
    iperf -c serverIP -d -t 530
  
After starting the server and the client, then run Cellsim on the Cellsim merchaine:
  sudo ./cellsim uplink_trace downlink_trace client_mac loss_rate 
  here we can set the loss_rate to 0 and can use tee to set the result to file.
  e.g.: 
  
    sudo ./cellsim ./att/uplink.pps ./att/downlink.pps 00:00:00:00:00:00 0 2>&1 | tee result/att_vegas.out
  
  Be aware that we should start at the same time and end the same time to evaluate different algorithms.
  
To analyze the result and generate the figure:
  To analyze the performance, we need three dimentions: Throughput, Utilization and 95% end-to-end self-inflicted delay.
  We can use two scripts to get them.
  scorer and quantiles (In the repo: https://github.com/keithw/sprout under src/examples)
  
  We can use the script as following, where the input file should be the output file of cellsim.
  
    cat <fileName> | ./scorer uplink | ./quantiles
    cat <fileName> | ./scorer downlink | ./quantiles
  
  Then we can get the output like:
    used: 180, available: 109275000
    Used 0 kbps / 7340 kbps => 0.000165 % 
    med: 59031, 95th: 112623
  Where the 95th is the 95% end-to-end delay.
  
  Then we need to calculate the minimum 95% end-to-end delay for a particular trace:
    The minimum 95% end-to-end delay is a property specific to a given cellular network trace.
    To compute it, take the corresponding input trace (available as .rx files at http://alfalfa.mit.edu/data/cleaned_traces.zip).
    The trace will have absolute UNIX timestamps in nanoseconds of the form 136....
    Convert these into relative timestamps in seconds since the beginning of the trace.
    After this conversion, you should have a file that describes packet delivery times in seconds relative to the beginning of the trace.
    To simulate an oracular zero delay, 100% throughput protocol on this trace, simply replace each timestamp "t" with :
    downlink t delivery 20
    or
    uplink t delivery 20
    depending on whether the trace is an uplink trace or a downlink trace.
    This replacement simulates a protocol which delivers packets exactly according to the schedule given in the trace, but with a fixed minimum delay of 20 ms per packet.
    In other words, this "oracular protocol" achives 100% throughput at no additional delay.
    Now, run either :
      cat <fileName> | ./scorer uplink | ./quantiles
      cat <fileName> | ./scorer downlink | ./quantiles
    This gives us the minimum 95% end to end delay say M for a particular cellular network trace.
    Ignore the message, "Illegal division by zero at ../../scorer line 51,"    
    Then use the 95% end-to-end delay to substract the the minimum 95% end to end delay, we can get 95% end-to-end self-inflicted delays.

  

