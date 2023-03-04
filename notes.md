# Developer Notes

A place to temporarily put notes until more permanent documentation is created.


## Building the Engine and Game

* Games are C++ projects that include the engine (optionally including the
  engine's graphical game editor).
* The editor generates C++ code from blueprint instances that is added to the
  C++ project.
* Premake is used to set up the C++ build environment (optionally including a
  project file for an IDE such as Visual Studio).
* A pre-build script is used to autogenerate C++ code when the project is built.
* The build environment selected in Premake is used to build the C++ code.


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


## Large File Repository

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


## Requirements and Testing Terms

The terms used in specifying requirements and in testing can be a bit ambiguous.
For the ExoG project, the following definitions are used. Some are project
specific.


### Terms for Specifying Requirements

<div style="margin-left: 3em; text-indent: -1em">

**feature** &mdash; a titled collection of related functional and non-functional
requirements used to describe some aspect of the system

<div style="text-indent: 0">

*\*\* This definition is project specific. \*\**

**Warning**: The term "feature" can be highly overloaded in general usage
because it does not specify the level of detail being discussed and because
features described in marketing can be different than those described in
technical documents. Be very careful when ecountering this word to make sure
that it is discussing a technical requirement according to the definition
given here.

</div>

**user epic** &mdash; a titled collection of related user stories

**user story** &mdash; Describes a user interaction with the system (possibly
through another system or piece of code) in terms of the goal of that
interaction

<div style="text-indent: 0">

Should use the form "So I can *\<reason for interaction>*, As a *\<description
of user's role>*, I want to *\<what the user wants to be able to do>*". "So I
can" might be replaced by a similar phrase such as "So as to be able to". User
stories should avoid implementation details and describe what is done, not how.
If the how is important, it should be added as a note outside of the main clause
with an explanation of its level of significance and why it is significant.

</div>

**user scenario** (or just "scenario") &mdash; a description of how the system
should behave as part of a user story

<div style="text-indent: 0">

Should use the form "Given *\<a set of initial conditions for the scenario>*,
When *\<action performed by the user>*, Then *\<the system's response to that
action>*". The "then" part of this clause should use the word "should" to
describe the correct response (e.g., "the application should respond a
warning"). User scenarios should avoid implementation details and describe what
is done, not how. If the how is important, it should be added as a note outside
of the main clause with an explanation of its level of significance and why it
is significant.

</div>

**user example** &mdash; a synonym for user scenario

<div style="text-indent: 0">

The term "user scenario" is somewhat preferred to avoid confusion with examples
produced for documentation.

</div>

**behavior** (alt. spelling "behaviour", but "behavior" is preferred) &mdash; A
synonym for either user story or user scenario

<div style="text-indent: 0">

Use of this term is discouraged except as part of the terms "behavioral test",
where it is preferred. It might also be used as part of the term "behavioral
requirement", but in this case, either the term "user story" or "user scenario"
should generally be used instead (which may require a rewording). The term
"functional requirement" might also be used together with the term
"non-functional requirement" when both are being discussed. In general, though,
it is better to discuss correct "behaviors" as defined by user stories and
scenarios than requirements.

</div>

**non-functional requirement** &mdash; a non-behavioral requirement of the
system such as its speed and memory usage

**functional requirement** &mdash; a behavioral requirement of the system

<div style="text-indent: 0">

Use of this term is highly discouraged in favor of the term "behavior" which is
defined in user scenarios (as a part of user stories), except when used in a
very broad context, particularly when used together with the term
"non-functional requirement".

</div>

</div>


### Terms for identifying the manner of testing

<div style="margin-left: 3em; text-indent: -1em">

**automated test** &mdash; tests that are executed automatically by the computer
at various stages of the deployment cycle or at the request of a user

**manual test** &mdash; tests that require users to interact with the system
to perform the test

<div style="text-indent: 0">

Manual tests that require the user to perform a specific sequence of steps to
check for a specific result should always be automated if they need to be
executed repeatedly.

</div>

**exploratory test** &mdash; manual tests done without a fixed set of steps to
explore some aspect of the systems behavior or the behavior as a whole

<div style="text-indent: 0">

These are generally done to ensure the behavior and the interface make sense.

</div>

</div>


### Terms for identifying the stage in the deployment cycle

<div style="margin-left: 3em; text-indent: -1em">

**commit test** &mdash; tests executed to validate commits to the main branch

**acceptance test** &mdash; tests executed after a commit to the main branch to
validate the build meets all acceptance criteria

**deployment test** &mdash; tests executed to verify a successful deployment of
the source or binaries

</div>


### Terms for identifying the purpose of the test

<div style="margin-left: 3em; text-indent: -1em">

**unit test** &mdash; tests performed on a single unit of code

<div style="text-indent: 0">

These are usually done as commit tests.

</div>

**integration tests** &mdash; tests used to verify requirements involving
multiple units of code

<div style="text-indent: 0">

These tests serve a variety of purposes at each stage of the deployment cycle.
Note that most behavioral tests and non-functional tests are types of
integration tests, but not all. The terms "behavioral test" and "non-functional
test" are preferred when referring to such tests.

</div>

**behavioral test** &mdash; Tests used to verify system behavior as sepcified
in user scenarios

<div style="text-indent: 0">

These are usually done as acceptance tests

</div>

**functional test** &mdash; A synonym for "behavioral test"

<div style="text-indent: 0">

The term "behavioral test" is strongly preferred

</div>

**non-functional test** &mdash; Tests used to verify non-functional requirements

<div style="text-indent: 0">

These are usually done as acceptance tests.

</div>

**code example** &mdash; A code example that serves both as documentation and a
test

<div style="text-indent: 0">

These require special markup for extraction to the documentation.

</div>

</div>


### Other terms used to describe tests

<div style="margin-left: 3em; text-indent: -1em">

**smoke test** &mdash; A test used to check for major problems and malfunctions
as representative of the system's overall health

<div style="text-indent: 0">

These are usually integration tests done to verify deployment or before other
tests in a suite to determine if it is even worth running the whole suite.

</div>

</div>
