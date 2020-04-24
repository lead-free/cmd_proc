# cmd_proc

A microcontroller friendly command processor.  

## Motivation  

A wide variety of embedded projects involve a command line interface. The idea is to create a light example of easilly managable command lookup mechanism. This project was inspired while handling the fololwing code:

```cpp
if(cmd == "led")
  led_set_power(arg)
    
if(cmd == "servo")
  servo_set_angle(arg)
  
//if(...)
```

This conditional tree approach becomes tidious to maintain when the number of executable commands grows. cmdproc offers an abstract, easilly tweakable comand lookup architecture.

## Howto

Let's look at the problem from abstract to specific. Consider the following code:

```cpp
while(true)
{
  // If there are commands in the usb queue -- execute them.  
  if(UsbComm::usb_queue.get_queue_size() != 0){

      const auto response = CommandParser::execute(UsbComm::usb_queue.get_next_cmd());

      // Respond to the command
      UsbComm::usb_send(&response);

      // Remove pending command from the queue
      UsbComm::usb_queue.free_pending_cmd();
  }
}
```

In the snippet above, received commands are handled by a seperate process (possibly interrupt) and put into a queue of terminated strings. Leave the reception details to the mysterious UsbComm. In this particular example, I was using a USB CDC driver; your commands can flow through any pheripheral interface (uart, spi, i2c, etc..).

Suppose the `UsbComm::usb_queue.get_next_cmd()` returns `"servo 180"`, which tells our microcontroller to set a servo to 180 degrees.

```cpp
const auto response = CommandParser::execute("servo 180");

UsbComm::usb_send(&response);
```

Here `CommandParser::execute("servo 180")` attempts to execute the given command and returns some meaningful status ("ok" or "unknown command" or "invalid args", etc..).
