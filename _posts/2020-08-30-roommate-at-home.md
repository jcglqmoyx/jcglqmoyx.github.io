# How do you know whether your roommate is at home or not?
>There are many ways to know whether your roommate is at home or not without actually bothering him. As a tech geek, I came up with a method to do this using a raspberry pi. This method doesn't guarantee reliability, but interestingly it works. Here's how it goes.
   
Let's assume that you are sharing the same Wi-Fi with your roommate, and you are able to login to the admin page of the Wi-Fi. Once you log in to the admin page, you will see how many devices have been connected to the Wi-Fi, what their names are and what their local area network IP addresses are.

After you log in to the admin site of the Wi-Fi, you will see what your roommate's phone's IP address is. In the case of mine, my roommate's IP address is 192.168.1.2. After getting his phone's IP address, I then was able to check whether his phone was online or not using the `ping` command on Linux.
>ping 192.168.1.2

And the response was like this:

    PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
    64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=51.4 ms
    64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=58.3 ms
    64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=84.10 ms
    64 bytes from 192.168.1.2: icmp_seq=4 ttl=64 time=105 ms
    64 bytes from 192.168.1.2: icmp_seq=5 ttl=64 time=25.0 ms
    ^C
    --- 192.168.1.2 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 9ms
    rtt min/avg/max/mdev = 25.019/64.856/104.647/27.567 ms

, which meant that his phone was online. So it was very likely that he was at home(and he was)!

Typing the `ping` command entails launching your computer and inputting the command to execute it every time you want to see whether your roommate's phone is online or not, which is way too troublesome. So let's 
simplify our work.

First, we write a Python script like this:

    import time
    ip = '192.168.1.2'
    back_info = os.system('ping -c 1 -w 1 %s' % ip)
    if back_info:
        print('Not at home')
    else:
        print('At home')

We then use the responding message of the `ping` command to control the on-off of a diode, which indicates whether your roommate' phone is online or not.
    import datetime
    import os
    import time
    
    import RPi.GPIO
    
    while True:
        time.sleep(1)
        ip = '192.168.1.2'
        back_info = os.system('ping -c 1 -w 1 %s' % ip)
        if back_info:
            print('not at home')
        else:
            print('at home')
            
            RPi.GPIO.setwarnings(False)
            RPi.GPIO.setmode(RPi.GPIO.BOARD)
    
            RPi.GPIO.setup(11, RPi.GPIO.OUT)
            RPi.GPIO.setup(12, RPi.GPIO.OUT)
    
            i = 1
            while True:
                RPi.GPIO.output(11, 1)
                RPi.GPIO.output(12, 1)
    
                time.sleep(1)
    
                RPi.GPIO.output(11, 0)
                RPi.GPIO.output(12, 0)
    
                i += 1
                if i == 10:
                    break
    
    RPi.GPIO.cleanup()

This is how you power your diode using raspberry pi's GPIO pins.
![avatar](https://pic4.zhimg.com/80/v2-509f3328a66393aecc029fedf91c42c3_1440w.jpg)
![avatar](https://picb.zhimg.com/80/v2-5e74d52ecca46f38dce4a13586f25016_1440w.jpg)

You then launch a terminal in the raspberry pi, enter the directory where your Python script(I named it roommate_at_home.py) is and type command `python roommate_at_home.py`. If your roommate is at home(actually you are checking whether his phone is connected to your apartment's Wi-Fi), the diode should begin to blink. Don't close the terminal and then use a wireless keyboard to connect to your raspberry pi. Press the up arrow key and the `Enter` key each time you want to run the program.

<a href='https://video.zhihu.com/video/1283537449365147648?'>Click here to see the demo video</a>
<video src="https://video.zhihu.com/video/1283537449365147648?" width="800px" height="600px" controls="controls"></video>