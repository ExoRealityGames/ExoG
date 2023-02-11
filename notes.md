## Objects

* Objects will have single inheritence from Object class
* In C++, use virtual methods as normal (this will allow C++ optimization of
  virtual calls when the type is known)
* Will later pull out implementations into private static methods for access by
  script to native thunks
* Script to native thunks will pull arguments off the script stack and place
  them on the native stack (if necessary) and call the implementation methods
  using templated function signatures
* Native to script thunks are the reverse and may or may not have a
  corresponding method on the C++ class (the pointer to the thunk may need to be
  looked up)


### Interfaces

* In C++, objects do not derive from the interface classes that they implement
  (this allows method names to be overridden)
* In C++, interface classes *always* use virtual base classes to derive from
  other interface classes
* Methods on the C++ interface wrap a call to a static method pointer
* Interfaces are registered with the type manager as part of type registration
* Clients request the interface to an object instance from the type manager
  through their own (the client's) type information (this allows the type
  manager to check if the client should have access to the interface)
* Scripts request interfaces in the same way as C++, but use the function
  pointers directly through templated function signatures


### Mixins

* Mixins are just interfaces with some methods implemented as part of the
  interface class (these methods may be virtual or non-virtual)
* Mixins may require specific interfaces on the type that they are being mixed
  into
* Mixins may only call methods on the interface class and those interfaces that
  they require


### V-Tables

* Objects of the same type share a common v-table that is pointed to by the
  instance
* Objects with interfaces have multiple v-table entries, one for each disjoint
  sequence
* Interfaces required by mixins are treated the same as if the mixing inherited
  from them for v-table construction
* Interface implementation methods must adjust the this pointer to get the
  object's instance data
* The static method that implements an interface will generally be unique for
  each method entry in the v-table, even if for the same interface, to allow the
  method to adjust the this pointer
* This complexity will eventually be hidden by automatic code generation, using
  small functions to adjust the this pointer and then call a user method, which
  will hopefully be inlined by the compiler if that is the only method calling
  the user method


## Large Files

* Large files will be stored as links that are pulled in by the build system
* The links are symbols to records within a data file that is stored in a
  separate repository (so the most recent can be used by earlier versions of the
  source in case files move)
* The records provide links to the actual file, whereever it is stored


## Configuration

* Platforms provide access to the project configuration
* The root platform is part of the build installed during deployment
* Other platforms (e.g., for running in the editor or testing) can be
  instantiated on top of the root platform by applications (e.g., the editor or
  test runner) and can therefore override the configuration provided
* Project configuration settings are grouped into configuration components
* Modules determine the components and settings available for configuration
* Platforms that require settings require a module to define those settings
  (this allows the settings to be present for configuration during development,
  even when the platform isn't the one being used)
* Some settings define the application context
  * Some settings are set per option from a component of the context
  * Some settings are specific to a single option from a component of the
    context
  * The settings for all contexts are stored as part of the configuration, but
    only those from the active context (determined at deployment or runtime) are
    used
  * The platform determines the active context at deployment or runtime
  * Application context settings can be nested within one another (e.g.,
    settings representing specific hardware inside a setting representing a
    platform)
  * The project settings for an application context option can inherit settings
    from the settings defined for another option of the context setting (e.g.,
    platform settings can have a set of settings fixed or defaulted across a
    range of platforms)
* Example components of the application context include:
  * Execution mode (e.g., batch test, test gui, editor, dedicated server, or
    client)
  * An environment type (e.g., development, testing, in editor, dedicated
    server, or client)
  * What operating system and hardware is present
* The bootstrapper creates an application domain using the settings from the
  project configuration according to the runtime context and then creates the
  application within that domain
* Consequently, the same application class can be used in multiple contexts
  with different settings
