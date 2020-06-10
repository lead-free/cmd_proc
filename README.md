# cmdproc

A microcontroller friendly command processor.  

## Motivation  

A wide variety of embedded projects involve a command line interface. The idea is to create a light example of an easily manageable command lookup mechanism. This project was inspired by handling code similar to:

```cpp
if(cmd == "led")
  led_set_power(arg)
    
if(cmd == "servo")
  servo_set_angle(arg)
  
//if(...)
```

This conditional tree approach becomes tedious to tweak when the number of executable commands grows. cmdproc offers an abstract, easily tweakable command lookup architecture.

## How To

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

In the snippet above, received commands are handled by a separate process (possibly interrupt) and put into a queue of terminated strings. Leave the reception details to the mysterious `UsbComm`. In this particular example, I used a USB CDC driver; your commands can flow through any peripheral interface (uart, spi, i2c, etc..).

Suppose the `UsbComm::usb_queue.get_next_cmd()` returns `"servo 180"`, which tells our microcontroller to set a servo to 180 degrees.

`const auto response = CommandParser::execute("servo 180");` [searches](CommandParser.cpp#L55) the *command table* for a matching *token* (in this example *token* is `"servo"`). In case a match is found, the corresponding command handler is [called](CommandParser.cpp#L67). Then space separeted arguments are parsed into a vector and passed to the command handler (in this example `"180"`). Finally, a meaningful status is returned ("ok", "unknown command", "invalid args", etc..).

### Command Table

Nothing fancy, just a [table](CommandParser.cpp#L15):
|Token|Help String|Commnad Handler|
|:---:|:---------:|:-------------:|
|led  |set led power [0-100]%|`HardwareDriver::led`|
|servo|set servo angle [0-180]deg|`HardwareDriver::servo`|

In practice, each row of the table above is an object of class [`CommandParser::Cmd`](CommandParser.h#L21) packed in a static array. We invoke the `HardwareDriver` that contains all abstracted command handlers to deal with the microcontroller's peripherals.

```cpp
std::array<Cmd, 2> command_table = {
  Cmd("led", "set led power [0-100]%", HardwareDriver::led),
  Cmd("servo", "set servo angle [0-180]deg", HardwareDriver::servo)
}
```

Adding a new command becomes one line of code added to the command table.

```cpp
Cmd("new_command", "adding new command is easy", HardwareDriver::unicorn)
```

## Known Issues

I am an aspiring developer and would be glad to hear your critique.

* In case the command table is long returning help_str as the whole table appended to one string might cause an overflow.
