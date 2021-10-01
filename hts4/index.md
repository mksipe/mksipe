<p style="text-align: center; font-size: 40px;"> HTS4</p>

---
# This is a WIP. It is not done.
## Project Documentation
- [Project Documentation](#project-documentation)
    - [System Documentation](#system-documentation)
        - [Requirements](#requirements)
            - [Operating System](#operating-system)
            - [Architechture](#architechture)
            - [Network Access](#network-access)
            - [Dependencies](#dependencies)
        - [Design Descisions](#design-descisions)
        - [Modifying a List](#modifying-a-list)
        - [Adding a New Section](#adding-a-new-section)
    - [User Documentation](#user-documentation)
        - [Installation](#installation)
            - [Manual](#manual)
            - [Automatic](#automatic)
        - [Usage](#usage)

### System  Documentation

#### Requirements

##### Operating System

The operating system must be a form of *nix or Linux. Git workflows have shown that *some* versions of windows allow this program to work. However, it is not natively supported. 


##### Architechture

The architecture used to create and run HTS4 has been on x86_64 based systems. However, Cargo tailors it to your system precisely.

##### Network Access

This program uses crates.io to carry out package retrieval and uses this to compile third-party crates (packages) and requires crates.io to be accessible to work correctly.

##### Dependencies

On the specified system to deploy this tool, you must have [Cargo](https://crates.io/) and rustc installed. Depending on your package manager and sources configured with your package manager, you should be able to install it through the specified dependency. i.e 

```sh
apt install cargo # debian 
pacman -syu cargo # arch
```

#### Design Descisions

The overall design of the program is to be easily modular. In the sense that, if you want to change it, you can do so.

I intentionally made this to search for specific things faster because of the overall amount of tools on a system that an entity may deploy. I may forget what a particular executable is. This program will find it, and I may either remember or look up the result and say: "ah I see."

Overall, this project is not meant to be professional nor intended to be hard to understand. The only things you need to understand are what is inside the cringelib and how the processes are being called.

The overall application is meant to be designed as follows:

There is the MAIN function in main.rs - This is required for rust to run. main.rs holds the command line arguments to execute from a terminal. Unless you add an entirely new list, function, or feature, you will not need to modify this file.

There is the CringeLib in the cringe/src directory - Which holds the actual searching algorithm. Every file in that directory is referenced to the lib.rs file, called in the main.rs file. Every other file contains a list of the services to search.

#### Modifying a List

I have intentionally made the file easy to add onto. Say there is a tool that you find that was supposed to be in the program but isn't. say, [hashcat](https://hashcat.net), for example.

The program is executable, so therefore I want to add it to the preexisting file named ext.rs. 

The file should look something like this:


```rs
use colored::*;

pub fn main() {
    println!("{}","\nExecutables".green());
    let list = vec![
        //list of executables to search for
        //I have omitted the already existing entries for simplification.

        ];
    for obj in list.iter() {
        let out = which::which(obj.clone());
        let _ret = match out {
            Ok(ref _ret) => println!("[{}] {:?}", "+".red(), out.unwrap()),
            Err(_error) => print!("{}", ""),
        };
    }
}

```

All necessary to add the program would be to add it in quotes, followed by a comma. 

```rs

use colored::*;

pub fn main() {
    println!("{}","\nExecutables".green());
    let list = vec![
        //list of executables to search for
        //I have omitted the already existing entries for simplification.
        "hashcat",
        ];
    for obj in list.iter() {
        let out = which::which(obj.clone());
        let _ret = match out {
            Ok(ref _ret) => println!("[{}] {:?}", "+".red(), out.unwrap()),
            Err(_error) => print!("{}", ""),
        };
    }
}

```

That is it. The file has already been referenced upon creation, so no further action is required. All necessary is to add the `-e` flag to the program upon usage after compilation.

#### Adding a New Section

Adding a new section is still straightforward. However, it is a bit trickier than simply adding an entry to an existing file. 

To add a whole new section, you will have to manually add the properties from claplib and reference it in the main function in main.rs.

Say you want to add a section to find web browsers with the executable flag identified as `-wb.`

You would first need to create the list with a name you will refer to the file as. For simplicity, I will be naming this file as webbrowser.rs. (There is a template to use called tmp.rs.tmp, which you can use. Just copy and rename the file to whatever you want to name it.)

After creating and adding entries to the file (refer to Modifying a List), you need to add it to the lib.rs file.

You can do this by adding (in my case):

`pub mod webbrowser`


Now that the file has been referenced as a library, you can add the command line functions to the program.

You need to modify the main.rs file to accomplish this.

The program uses traditional claplib syntax. All you need to do is follow it [clap-rs](https://github.com/clap-rs/clap).

After adding your CLI options to be recognized by clap, you need to add the specified match statement to two sections:

you need to create your own which will be (as per my example):

```rs
    if matches.is_present("wb"){
        cringe::webbrowser::main();
    }
```

You will also need to add it in the `find-all` section anywhere in the if statement next to the existing entries.

You should have a functioning custom CLI option.


### User Documentation

#### Installation

Please revert to the supported operating systems before trying to make an install.

##### Manual

To make a manual installation, you need to download the [software](https://github.com/mksipe/hts4) via git or through an HTTP download.

***
>###### git

 ```sh
 git clone https://github.com/mksipe/hts4
 cd hts4
 ```
***
>###### http
Click the green code button, 
select Download ZIP

```
cd /home/$USER/Downloads
tar xvf hts4-main.zip
cd hts4
```
***

With cargo already installed, all you need to do is use cargo to run the program.

`cargo run -- <args>`

##### Automatic

An automatic installation will install the program to your system and place it in the /usr/bin directory
> Install.ps1 does not work!

After installing the program, all that is required is to do the following:

> change executable permissions

`chmod 700 Install.sh`

> make install

`sudo ./Install.sh`

After the install, the source directory may be deleted

#### Usage

By default the following are built into the program.

|Args|Description|
|-|-|
|-e|Finds all executables that match the built in list|
|-s|Finds services that are installed on a specific system|
|-g|Finds game packages that may have been installed on a system|
|-w|Searches for web binaries that have networking functionalty|
|-i|Searches for language interprets on the system.|
|-a|Runs all searches|
---

