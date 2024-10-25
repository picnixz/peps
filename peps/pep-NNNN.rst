PEP: NNNN
Title: Exception Group Matching and Splitting Semantics on Leaf Exceptions
Author: Bénédikt Tran
Sponsor: N/A
PEP-Delegate: N/A
Discussions-To: N/A
Status: Draft
Type: Standards Track
Created: 24-Oct-2024
Python-Version: 3.14
Post-History: N/A
Resolution: N/A


Abstract
========

This document proposes language extensions that allow programs to match
and split leaf exceptions as if they were exception groups instead of
wrapping them in an exception group.


Motivation
==========

Exception groups allow a program to raise and propagate unrelated exceptions.
While it is possible to handle exception groups and leaf exceptions separately
using ``except*`` and ``expect`` clauses respectively, inspecting an exception
beyond its type may require code duplication as illustrated by the following
example:

.. code-block:: python

   def is_error_404(exc):
       return isinstance(exc, HTTPError) and exc.status_code == 404

   try:
       res = do_http_request()
   except ExceptionGroup as err:
       not_found = err.subgroup(is_error_404) is not None
   except HTTPError as err:
       not_found = err.status_code == 404


Rationale
=========

[Describe why particular design decisions were made.]


Specification
=============

We propose to add two methods on :class:`BaseException`, namely ``subgroup``
and ``split`` which are equivalent to the following Python code:

.. code-block:: python

   class BaseException:
       def subgroup(self, condition):
           return BaseExceptionGroup(str(self), [self]).subgroup(condition)

       def split(self, condition):
           return BaseExceptionGroup(str(self), [self]).split(condition)

Stated otherwise, matching and splitting a leaf exception is equivalent to
wrap that exception in an exception group before calling the corresponding
methods.


Backwards Compatibility
=======================

None.


How to Teach This
=================

[How to teach users, new and experienced, how to apply the PEP to their work.]


Reference Implementation
========================

The reference implementation is [#1]_.


Rejected Ideas
==============

A utility function
------------------

Introducing a utility function to transform a leaf exception to a group
exception was considered:

.. code-block:: python

   def as_group(exc):
       if isinstance(exc, BaseExceptionGroup):
           return exc
       return BaseExceptionGroup("", [exc])

Its usage would then be:

.. code-block:: python

   def is_error_404(exc):
       return isinstance(exc, HTTPError) and exc.status_code == 404

   try:
       res = do_http_request()
   except Exception as exc:
       not_found = as_group(exc).subgroup(is_error_404)

Nevertheless, this requires maintaining the helper across different Python
versions and follow the semantics of exception groups. In addition, adding
an intermediate helper slightly reduce the readability.

A method to convert a leaf exception to a group
-----------------------------------------------

Similarly, a method on :class:`BaseException` to wrap a leaf exception in
a :class:`BaseExceptionGroup` consisting of that exception was considered
yet rejected since it would add a new method on both leaf exceptions and
exception groups.

The existence of the :meth:`~BaseExceptionGroup.split` and the
:meth:`~BaseExceptionGroup.subgroup` on exception groups motivated
the choice of exposing a similar interface on :class:`BaseException`.


Footnotes
=========

.. [#1] https://github.com/python/cpython/pull/125883


Copyright
=========

This document is placed in the public domain or under the
CC0-1.0-Universal license, whichever is more permissive.
