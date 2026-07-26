# Dumb Editor

I set up the C++ and required files, installed build tools, CMake, Compliers

> I setup the basic file structures 
> In header files we write the abstract classes and in .cpp files we define those class's methods

```txt
app\
|	|-- main.cpp
|
core\
|	|-- cursor.h
|	|-- cursor.cpp
|	|-- document.h
|	|-- document.cpp
|	|-- editor.h
|	|-- editor.cpp
|
tests\
|
CMakeLists.txt
|
README.md
```

basically editor controls both cursor and document 
document handles loading and saving the document and cursor takes care of movement of the cursor. Cursor talks to editor and editor handles with document

### Why Cmake ?

```txt
cmake_minimum_required(VERSION 3.20)
project(editor)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

  
add_executable(editor
    app/main.cpp
    core/editor.cpp
    core/document.cpp
    core/cursor.cpp
)
```

this txt files make it possible compile large projects with lots of .cpp files 
if we have lots of files we have to include them in our command one by one instead we can list it here it will automatically read this file when 

```cmd
cmake -S <source_folder> -B <build_folder>
```

this will read the source folder for the above txt file and write the command for building the project 

then,

```cmd
cmake --build <build_folder>
```

will create the executable file to run

# Learnings
- `size_t is unsigned type used in cursor to avoid negative indexes`
- `setter's and getter's are used to keep private attributes not assisible from outside`
- `'void func() const'- const keyword after function definition provides contract to compiler that no operation of writing to a variable will happen inside`
``

