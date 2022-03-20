**Core SDK candidate assignment**

Solution:

The original project structure had strong connection with platform implementation of http functionality. Talking ahead, I was not able to completely get rid of dependency between http static library within my solution with core library.

My implementation of http static library is located in `platform/http`. The general interface of http static library is located in `platform/http/http_file_source.hpp` (simply moved from storage component source files as individual component to http library). You can find library definition via CMake file in `platform/http/CMakeLists.txt`.

Currently, http library has big dependency on core library. Getting rid of dependency requires implementation rework. At least it is needed to get rid of usage of utility functions of mbgl-core im platform implementations or collect all common utility functionality and bind it together as static library that will be used in mbgl-core and mbgl-http libraries.

For future direction I would suggest reimplement each component in core library as individual static library (maybe even add them as git submodules). The core itself would depend on each component and link with them statically. Each component should not depend on each other (something like Qt that has widgets, gui, network and etc.). The reason behind separating each component is to prevent possible implementation dependency between components (better control of dependency management within mbgl-core).
