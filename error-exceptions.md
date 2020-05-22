====== PHP RFC: Errors as Exceptions ======
  * Version: 0.1
  * Date: 2020-05-22
  * Author: Katie Volz, iggyvolz@gmail.com
  * Status: Draft
  * First Published at: http://wiki.php.net/rfc/error_exception


===== Introduction =====
Currently, many errors in PHP lead to an Error being triggered.  Errors, unlike exceptions, do not break out of the execution context and will either cause PHP to exit fatally, or silently log the error and continue.  As this is undesirable for many programs, the following snippet from the ErrorException documentation (or something like it) is often executed to promote these to Exceptions:

<code php>
function exception_error_handler($severity, $message, $file, $line) {
    if (!(error_reporting() & $severity)) {
        // This error code is not included in error_reporting
        return;
    }
    throw new ErrorException($message, 0, $severity, $file, $line);
}
set_error_handler("exception_error_handler");
</code>

However, this approach has several issues:
* This requires a global bootstrap to be executed, which may be extra overhead in some environments (ex. unit testing)
* The error handler cannot be set in library code without affecting other libraries
* Backtraces refer to the error handler function rather than the line at which the error occurred
* This requires an extra function overhead for both initialization and error handling

This RFC proposes an opt-in declare statement (similar to declare_types) to override the default error behaviour with an exception being thrown in all cases except compile errors (which are not possible to handle in userspace).

===== Proposal =====
This RFC introduces a new ''declare()'' directive, ''error_exception'', which takes either ''1'' or ''0''.  If ''1'' is passed, any error occurring in this file (TODO: which errors) will be promoted to an ErrorException.  If ''0'' is passed (the default if the statement is not declared), the error will be triggered as the current behaviour.

===== Backward Incompatible Changes =====
As ''error_exception'' defaults to 0, there are no backward incompatibilities. 

===== Proposed PHP Version(s) =====
Next minor version (8.0).

===== RFC Impact =====
==== To SAPIs ====
None.

==== To Existing Extensions ====
TODO: Is the change in zend_error_va_list enough to affect extensions?

==== To Opcache ====
None.

===== Open Issues =====
* The name error_exception does not clearly describe the behaviour of this switch

===== Unaffected PHP Functionality =====
Any error triggered during compile time continues to operate normally, as we have not entered PHP execution.

===== Future Scope =====
Potential future improvements (not part of this RFC) include:
* Creating separate classes for different error levels so they can be caught differently (ex. NoticeErrorException)
* More specific errors for particular function when error_exception is 1 (ex. undefined variables might throw an UndefinedVariableException if error_exception is 1, or throw a warning if error_exception is 0)

===== Proposed Voting Choices =====
* Should the declaration as defined in this RFC be accepted? (2/3)
* What should be the name of the declaration statement be? (majority of: error_exception/strict_errors/error_exceptions/???)

===== Patches and Tests =====
Prototype patch (partially complete): https://github.com/iggyvolz/php-src/tree/strict_errors