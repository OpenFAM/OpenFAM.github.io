# Error handling

Note that unlike C or C11, C++ provides native support for exceptions. Hence the C++ API adopts the C++ style of handling
and reporting errors to its caller, using exceptions. Unlike the normal C convention of returning integer values (0 for success
and negative values in case of failure), it uses C++ exceptions to indicate error scenarios. All APIs return the expected output
on success (or void if no output is returned), and throw an exception on error. Instead of checking the return value for errors
as in C, the caller should use try-catch blocks for handling errors.
  
The implementation defines a set of exception classes, which are derived from C++ standard exception class to categorize
various types of failures, like Runtime, Allocator, Data-path, etc. It also defines a list of individual error numbers to
identify a specific error. The exception object received by the caller in case of an error will contain a specific error
number and an appropriate error message string. The application can retrieve this information using member functions,
fam\_error() and fam\_error_msg()/what() and take any necessary action. The currently defined exceptions (and exception numbers)
are defined in Table 1 and Table 2.

<table>
<caption>Table 1: List of OpenFAM exceptions</caption>
  <tr><th>Fam Exception</th> <th>Class Description</th></tr>
  <tr><td>Fam_Exception</td> <td>This is the base exception class for all fam exceptions.</td></tr>
  <tr><td>Fam_InvalidOption_Exception</td> <td>For invalid options or arguments for fam APIs.</td></tr>
  <tr><td>Fam_Timeout_Exception</td> <td>Retry or Timeout limit for blocking APIs.</td></tr>
  <tr><td>Fam_Datapath_Exception</td> <td>Indicate data path errors.</td></tr>
  <tr><td>Fam_Allocator_Exception</td> <td>Indicate allocator errors.</td></tr>
  <tr><td>Fam_Pmi_Exception</td> <td>Indicate runtime initialization errors.</td></tr>
  <tr><td>Fam_Unimplemented_Exception</td> <td>Calling unimplemented OpenFAM APIs.</td></tr>
</table>

<table>
<caption>Table 2: List of OpenFAM error numbers</caption>
<tr><th>Fam Error</th><th>Description</th></tr>
<tr><td>FAM_ERR_UNKNOWN</td><td>Unexpected or Unknown errors.</td></tr>
<tr><td>FAM_ERR_NOPERM</td><td>Caller does not have access rights for the desired operation.</td></tr>
<tr><td>FAM_ERR_TIMEOUT</td><td>Blocking APIs reached retry/timeout limit.</td></tr>
<tr><td>FAM_ERR_INVALID</td><td>APIs called with invalid options/arguments.</td></tr>
<tr><td>FAM_ERR_LIBFABRIC</td><td>Libfabric API failure.</td></tr>
<tr><td>FAM_ERR_SHM</td><td>Shared memory allocator error.</td></tr>
<tr><td>FAM_ERR_NOTFOUND</td><td>Data item or region not found in FAM.</td></tr>
<tr><td>FAM_ERR_ALREADYEXIST</td><td>Data item or region already exists in FAM.</td></tr>
<tr><td>FAM_ERR_ALLOCATOR</td><td>Allocator specific error</td></tr>
<tr><td>FAM_ERR_GRPC</td><td>Error from grpc layer.</td></tr>
<tr><td>FAM_ERR_PMI</td><td>Runtime error.</td></tr>
<tr><td>FAM_ERR_OUTOFRANGE</td><td>Data access out of range.</td></tr>
<tr><td>FAM_ERR_UNIMPL</td><td>Calling unimplemented functions/APIs.</td></tr>
<tr><td>FAM_ERR_RESOURCE</td><td>Resource not available.</td></tr>
<tr><td>FAM_ERR_INVALIDOP</td><td>Invalid operations</td></tr>
</table>

Note that the library contains both blocking and non-blocking calls for most data path operations.
In case of errors, all blocking calls throw exceptions immediately. For example, a call to fam_put_bocking()
will either complete successfully, or throw one of the following exceptions - Fam_InvalidOption_Exception,
Fam_Allocator_Exception, Fam_Datapath_Exception or Fam_Timeout_Exception. In general, the application
should use the normal try-catch block to handle exceptions:

```C++
try { 
	fam_put_blocking();
} catch (Fam_Exception &e) {
	// Exception handling code
}
```

Note that the library contains both blocking and non-blocking calls for most data path operations. In case of errors, all blocking calls throw exceptions immediately. For example, a call to fam\_put\_bocking() will either complete successfully, or throw one of the following exceptions - ```Fam\_InvalidOption\_Exception```, ```Fam\_Allocator\_Exception```, ```Fam\_Datapath\_Exception``` or ```Fam\_Timeout\_Exception```. In general, the application should use the normal try-catch block to handle exceptions:

However, the non-blocking calls are queued within the library, and may not catch exceptions immediately. Depending on the underlying error, certain exceptions will be thrown immediately, while others may only be thrown during the next fam_quiet() call. Thus the code may look like:

```C++
try { 
	fam_put_nonblocking();
} catch (Fam_Exception &e) {
	// handle (for example) Fam_InvalidOption_Exception,
	// Fam_Allocator_Exception, Fam_Datapath_Exception
}
	// … Continue rest of the code …
try {
	fam_quiet();
} catch (Fam_Exception &e) {
	// These exceptions may actually result from a previous fam_put_nonblocking() operation
	// Fam_Timeout_Exception, Fam_Datapath_Exception
}
```

Note that uncaught exceptions will result in the application being terminated as if a fam\_abort() had been called.
