---
title: "Xored"
date: 2022-08-23T13:33:28-05:00
draft: true
---

# XORed

## This is a small tool I made to perform XOR operations on a binary file using Golang.

[Find it at my github: xored.go](https://github.com/itavares/golang-tools/blob/main/xored.go)

### The motive to create this tool (it's very simple honestly) happened while solving a threat hunting CTF problem where I was tasked to find the binary in a pcap file. The neat part was that the binary was "hidden." The attacker performed an XOR operation to hide it, so to get the original binary, I would have to undo the operation.

## The Code:

- xored.go
```golang
/*
Author: Ighor Tavares (@sum_b0dy)
Date: 23 August 2022
This program is used to perform XOR operation on binary files (read byte array and perform xor given key)
*/

package main

import (
	"flag"
	"fmt"
	"os"
)

func main(){

	inputFile := flag.String("in","","Input File. (ie. example.exe)")
	outputFile := flag.String("out","","Output File. (ie. output.exe)")
	flag.Parse()

	//XOR key
	key := 0xAA

	//read the file
	b, errR := os.ReadFile(*inputFile)
	if errR != nil{
		fmt.Fprintf(os.Stderr, "xored: %v\n", errR)
		os.Exit(1)
	}

	//type byte == uint8 (byte is alias to uint8)
	// fmt.Printf("%T\n",b)
	// fmt.Printf("%X\n", key)
	// fmt.Printf("%d\n",uint8(key))

	//make array of byte to store the XORed bytes
	var byteArray []uint8 

	for _, bt := range b {
		xorOp := bt ^ uint8(key)
		byteArray = append(byteArray, xorOp)
	}

	//Write the byteArray to the new binary with given output name from flag
	errW := os.WriteFile(*outputFile, byteArray, 0644)
	if errW != nil{
		fmt.Fprintf(os.Stderr, "xored: %v\n", errW)
		os.Exit(1)
	}
}
```

## Demo

I will use an example like the CTF challenge I used this script on. We have an executable in which XOR was performed on. We are given an XOR key, and we suppose to use it to undo the XOR operation on the binary.

```
Usage of xored.exe:
  -in string
        Input File. (ie. example.exe)
  -out string
        Output File. (ie. output.exe)
```

This binary we find contains a lot of 0xaa bytes, which is an indication of XOR operation. This is because when a null byte is XORed with a key (0xaa), the null byte becomes the key itself:

![Example image](/xored-image1.png)

We can see all the AA's in this binary. Now, using the script, we can undo that operation with a known key (in this case we are given that is 0xaa).

![Example image](/xored-image2.png)

Now, if we examine the output.exe file we created after the XOR operation, we can see that it reverts to the original binary (based on the headers).

![Example image](/xored-image3.png)



