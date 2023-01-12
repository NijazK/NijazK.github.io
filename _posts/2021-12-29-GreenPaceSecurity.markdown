---
layout: post
title:  "Green Pace Security Development (C++)"
date:   2021-12-29 14:34:25
categories: jekyll update
tags: 
image: /assets/article_images/2021-29-12-GreenPaceSecurity/GreenPaceSecurity.jpg
---

## Secure coding Practices using C++.

### Overview
Software development at Green Pace requires consistent implementation of secure principles to all developed
applications. Consistent approaches and methodologies must be maintained through all policies that are
uniformly defined, implemented, governed, and maintained over time.

### Purpose
This policy defines the core security principles; C/C++ coding standards; authorization, authentication, and
auditing standards; and data encryption standards. This article explains the differences between policy,
standards, principles, and practices (guidelines and procedure): Understanding the Hierarchy of Principles,
Policies, Standards, Procedures, and Guidelines.

### Scope
This document applies to all staff that create, deploy, or support custom software at Green Pace. In short, these ten principles were adopted to create a successful security software architecture.

    1. Validate Input Data.
    2. Heed Compiler Warnings.
    3. Architect and Design for Security Policies.
    4. Keep it Simple.
    5. Default Deny.
    6. Adhere to the Principle of Least Privilege.
    7. Sanitize Data Sent to Other Systems.
    8. Practice Defense in Depth.
    9. Use Effective Quality Assurance Techniques.
    10. Adopt a Secure Coding Standard.

### 10 Coding Practices
A snapshot of the template used by the team to produce the ten coding standard practices.
![image](https://user-images.githubusercontent.com/75659218/211977464-a0ec8540-3b77-437a-b020-b2c629222068.png)
Noncompliant Code
![image](https://user-images.githubusercontent.com/75659218/211977503-1a77d223-9b90-4bc2-909b-febd3c8344b5.png)
Compliant Code
![image](https://user-images.githubusercontent.com/75659218/211977554-ed6d203c-3261-46dd-9804-19458f01f4eb.png)
Threat Level
![image](https://user-images.githubusercontent.com/75659218/211977586-65d2d0b6-c512-4df8-9b40-b33c6ec00f10.png)
Automation
![image](https://user-images.githubusercontent.com/75659218/211977607-27b686d7-9db2-418b-8473-580927554f12.png)

10 Coding Standards (c+++
![image](https://user-images.githubusercontent.com/75659218/212163281-683633a8-cf4f-40ee-98e8-9b2ab982bdd9.png)


### Encryption   

    1. Encryption in rest.
      * Encryption for data at rest is the process of securely encoding data as it is written into storage and
        decrypting that data as it is pulled from storage for use
    2. Encryption at flight.
      * Encryption of data in-flight is the process of securely encoding data as it is being transmitted in
        some fashion.
    3. Encryption in use.
      * Encryption of data in-use is the process of protecting data as it is utilized in memory, the main
        way of doing this is by utilizing password protected profiles as they protect the memory of each
        user for the data stored in memory for that profile could be used to compromise their data in
        rest/flight.

The code snippet below is from the main file that will encrypt or decrypt a string using the provided key.
    
    // Encryption.cpp : This file contains the 'main' function. Program execution begins and ends there.
    // Author: Nijaz Kovacevic
    // Encryption for Green Pace Security

    #include <cassert>
    #include <fstream>
    #include <iomanip>
    #include <iostream>
    #include <sstream>
    #include <ctime>
    #include <time.h>
    #pragma warning(disable : 4996)

    /// <summary>
    /// encrypt or decrypt a source string using the provided key
    /// </summary>
    /// <param name="source">input string to process</param>
    /// <param name="key">key to use in encryption / decryption</param>
    /// <returns>transformed string</returns>
    std::string encrypt_decrypt(const std::string& source, const std::string& key)
    {
        // get lengths now instead of calling the function every time.
        // this would have most likely been inlined by the compiler, but design for perfomance.
        const auto key_length = key.length();
        const auto source_length = source.length();

        // assert that our input data is good
        assert(key_length > 0);
        assert(source_length > 0);

        std::string output = source;

        // loop through the source string char by char
        for (size_t i = 0; i < source_length; ++i)
        { // TODO: student need to change the next line from output[i] = source[i]
          // transform each character based on an xor of the key modded constrained to key length using a mod
            output[i] = source[i] ^ key[i % key_length];
        }

        // our output length must equal our source length
        assert(output.length() == source_length);

        // return the transformed string
        return output;
    }

### Triple A Framework

      1. Authentication.
        * The process used to prove who a user is by, userID, passwords, possibly higher-level security
          such as secure tokens, CAC/PIN and other hardware credentials.
      2. Authorization.
        * Once a user is authenticated and allowed access to a system, they are granted specific access to
          parts of that system. Authorized access to certain drives, folders, programs, or data allowed by
          the system administrators.
      3. Accounting.
        * After authentication and authorization, it is always a good idea to monitor and record activity of
          all users on the system.

### Audit Controls and Management
Every software development effort must be able to provide evidence of compliance for each software deployed
into any Green Pace managed environment.
Evidence will include the following: Code compliance to standards, Well-documented access-control strategies, with sampled evidence of compliance, Well-documented data-control standards defining the expected security posture of data at rest, in flight, and in use, Historical evidence of sustained practice (emails, logs, audits, meeting notes).

### Enforcement 
The office of the chief information security officer (OCISO) will enforce awareness and compliance of this
policy, producing reports for the risk management committee (RMC) to review monthly. Every system
deployed in any environment operated by Green Pace is expected to be in compliance with this policy at all
times. Staff members, consultants, or employees found in violation of this policy will be subject to disciplinary action,
up to and including termination.

### Exceptions Process
Any exception to the standards in this policy must be requested in writing with the following information: Business or technical rationale, Risk impact analysis, Risk mitigation analysis, Plan to come into compliance, Date for when the plan to come into compliance will be completed. Approval for any exception must be granted by chief information officer (CIO) and the chief information security officer (CISO) or their appointed delegates of officer level. Exceptions will remain on file with the office of the CISO, which will administer and govern compliance.

### Distribution
This policy is to be distributed to all Green Pace IT staff annually. All IT staff will need to certify acceptance
and awareness of this policy annually.

### Policy Change Control
This policy will be automatically reviewed annually, no later than 365 days from the last revision date. Further,
it will be reviewed in response to regulatory or compliance changes, and on demand as determined by the
OCISO.
