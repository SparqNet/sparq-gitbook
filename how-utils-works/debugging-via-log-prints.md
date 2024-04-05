# Debugging via log prints

Due to the Subnet's behavior when it is initialized (`subnet->start()`), printing details in the terminal via `std::cout` doesn't work as it should. To properly debug the Subnet, you have to use the functions `Utils::logToFile()` and `Utils::LogPrint()`.

The function `Utils::logToFile()` logs to the Node's `log.txt` file and requires only the string that will be logged. The function `Utils::LogPrint()` logs to the Node's `debug.txt` file and requires the string that will be logged, the function name (you can use the `__func__ macro` here) and the module prefix from where the function is being called, that matches an item from the Log namespace (e.g. "`Log::Subnet = 'Subnet::'`").\


Example:

```cpp
Utils::logToFile("Logging");
// log.txt will say "Logging"

void testFunc() {
  Utils::LogPrint(Log::db, __func__, "Debugging");
  // debug.txt will say "DBService::testFunc: Debugging"
}
```
