.. _ref-c-api:

C API for Handwritten Code
==========================

The :mod:`sip` module provides an API that can be used by handwritten code in
specification files.  The API has a number of versions each of which is
implemented by an ABI.  ABIs use semantic versionion so that, for example,
v12.1 is compatible with v12.0 but is incompatible with v11.0, v12.2 and v13.0.

The API is declared in the :file:`sip.h` header file.  SIP will ensure this
file is installed in a location such that the compiler will automatically find
it.

The details of each currently supported ABI are described in subsequent
sections.  The remainder of this section covers topics that are common to all
ABI versions.


.. _ref-type-structures:

Generated Type Structures
-------------------------

SIP generates an opaque type structure for each C structure, C++ class, C++
namespace, traditional named enum or mapped type being wrapped.  These are
:c:type:`sipTypeDef` structures and are used extensively by the API.

The names of these structure are prefixed by ``sipType_``.

For those structures that correspond to C structures, C++ classes, C++
namespaces or traditional named enums the remaining part of the name is the
fully qualified name of the structure, class, namespace or enum name.  Any
``::`` scope separators are replaced by an underscore.  For example, the type
object for class ``Klass`` is ``sipType_Klass``.

For those structure that correspond to mapped types the remaining part of the
name is generated by SIP.  The only way for handwritten code to obtain a
pointer to a structure for a mapped type is to use :c:func:`sipFindType()`.

The type structures of all imported types explicitly used by a module are
available to handwritten code.


.. _ref-derived-classes:

Generated Derived Classes
-------------------------

For most C++ classes being wrapped SIP generates a derived class with the same
name prefixed by ``sip``.  For example, the derived class for class ``Klass``
is ``sipKlass``.

If a C++ class doesn't have any virtual or protected methods in it or any of
it's super-class hierarchy then a derived class is not generated.

Most of the time handwritten code should ignore the derived classes.  The only
exception is that handwritten constructor code specified using the
:directive:`%MethodCode` directive should call the derived class's constructor
(which has the same C++ signature) rather then the wrapped class's constructor.


.. _ref-exception-objects:

Generated Exception Objects
---------------------------

SIP generates a Python object for each exception defined with the
:directive:`%Exception` directive.

These objects are named with the fully qualified exception name (i.e. including
any enclosing scope) prefixed by ``sipException_``.  For example, the type
object for exception ``Except`` defined in class ``Klass`` is
``sipException_Klass_Except``.

The objects of all imported exceptions are available to handwritten code.


Event Handlers
--------------

The :mod:`sip` module will trigger a number of events.  Handwritten code can
supply handlers for these events to allow it to perform additional actions.
Each event has a type, described by the :cpp:enum:`sipEventType` enum.  An
event handler is registered using :c:func:`sipRegisterEventHandler()`.  The
signature of an event handler is specific to the event type.


Using the C API when Embedding
------------------------------

The C API is intended to be called from handwritten code in SIP generated
modules.  However it is also often necessary to call it from C/C++ applications
that embed the Python interpreter and need to pass C/C++ instances between the
application and the interpreter.

The API is exported by the :mod:`sip` module as a ``sipAPIDef`` data structure
(defined in the :file:`sip.h` header file) containing a set of function
pointers.  The data structure is wrapped as a Python ``PyCapsule`` object.  It
is referenced by the name ``_C_API`` in the :mod:`sip` module's dictionary.

Each member of the data structure is a pointer to one of the functions of the
C API.  The name of the member can be derived from the function name by
replacing the ``sip`` prefix with ``api`` and converting each word in the
name to lower case and preceding it with an underscore.  For example:

    ``sipExportSymbol`` becomes ``api_export_symbol``

    ``sipWrapperCheck`` becomes ``api_wrapper_check``

Note that the type objects that SIP generates for a wrapped module (see
:ref:`ref-type-structures` and :ref:`ref-exception-objects`) cannot be refered
to directly and must be obtained using the :c:func:`sipFindType()` function.
Of course, the corresponding modules must already have been imported into the
interpreter.

The following code fragment shows how to get a pointer to the ``sipAPIDef``
data structure::

    #include "sip.h"

    const sipAPIDef *get_sip_api()
    {
        return (const sipAPIDef *)PyCapsule_Import("sip._C_API", 0);
    }

``"sip._C_API"`` should be replaced by the fully qualified name of the
:mod:`sip` module.

If you need to install a local copy of the :file:`sip.h` header file then you
can do so with :program:`sip-module`.
