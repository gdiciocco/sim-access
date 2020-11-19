# sim-access
A Python module for managing SIM modules - this is a fork of the original from Wilson Wang which is one of the best libraries I found for this application.


## Setup
The setup is:
- SIM868 from waveshare connected via GPIO to a raspberry pi zero w

## Changes
get__raw_rssi() - returns a tuple containing raw level and ber according to gsm module  

get_rssi() - returns a tuple returning signal level in str "-nn dBm" and BER converted to quality str "high" "medium" or "low"

sms_delall() - used to empty all the message storages 

GPS disabled during initialization sequence, you can enable it back.

Added AT+CSCLK=0 in initialization sequence to disable sleep.

Increased counter of wait_ok to 6 as it was giving false negatives sometimes, needs to be understood and refined.

I've connected PWRKEY to a custom gpio and i'm handling it from the main app, however it could be already available somewhere, i'll integrate it in the library eventually. As the module sometimes needs to be power cycled and/or reset and there's a reset pin too, according to the sim868 docs. Got to take a deeper look in the future.


## Coding

### Texts and Calls

To receive SMS and calls, you need to write a class from base class SIMModuleBase. There are two method you need to implement. **on_sms()** and **on_call()**. Here is one example.

```python
class MySIM(SIMModuleBase):
    def on_sms(self, number, content):
        print('Text from: {0}, Content: \"{1}\"'.format(number, content))

    def on_call(self, number):
        print('Got phone call from {0}'.format(number))
        time.sleep(5)
        self.call_hangup()

    def on_call_missed(self, number):
        ''' This function is not working for SIM800
        '''
        self.sms_send(number, 'Sorry, I missed your call!')

if __name__ == '__main__':
    MySIM().mainloop()

```

You can detach the mainloop thread

``` python
    a.mainloop(True)
```

You can manage the loop yourself

``` python
    while True:
        # do something 1 ...
        a.loop_once()
        # do something 2 ...
```

### GPS

You can also get your GPS locations, date and time using method **gps_location_date_time**

``` python
    class MySIM(SIMModuleBase):
        ...
    
    sim = MySIM()
    ((mylong, mylat), mydate, mytime) = sim.gps_location_date_time('<YOUR APN>')
    print('Longitude: {0}\nLatitude: {1}\nDate: {2}\nTime: {3}\n'.format(mylong, mylat, mydate, mytime))
```


Whenever you received an SMS, **on_sms()** willl be called. If you receive a phone call, **on_call()** will be called. Please note that **on_call()** could be called multiple times during a phone call.

There is no implemenation of answering the phone call right now. The SIM module I bought does not support answering phone calls.

## Implementation

Internally, I use a thread to monitor incoming texts and calls.
