# SecureMemory

SecureMemory is a simple library that provides a secure memory allocation and deallocation functions. It is designed to be used in applications that require secure memory handling, such as password managers, encryption tools and other security-critical applications.

Under the hood, SecureMemory uses the `mlock` system call to lock the memory pages into RAM, preventing them from being swapped to disk. This is useful when dealing with sensitive data, as it ensures that the data is never written to disk, even if the system runs out of memory.
Data can be XORed with a random key after being written to memory, and the key can be securely wiped from memory after use.

For more protection of super-sensitive data, you can protect the memory from being read/written/executed or just
remove all permissions from the memory, so it will not be accessible at all until you give the permission back.

## Installation
Please note that SecureMemory is only supported on UNIX systems (Linux, macOS and etc).

Maven:
```xml
<dependency>
  <groupId>org.exploit</groupId>
  <artifactId>secure-memory</artifactId>
  <version>1.0</version>
</dependency>
```
Gradle:
```groovy
implementation 'org.exploit:secure-memory:1.0'
```

## Usage

### Basic memory allocation with obfuscation and deallocation
It will be enough for most of the cases, when we store sensitive data.
We can allocate any size of memory, write data to it and obfuscate it.

Data will not be swapped to disk (that is critical for sensitive data)
```java
public class SecureMemoryExample {
    public static void main(String[] args) {
        byte[] data = "Hello, World!".getBytes();
        byte[] anotherData = "Secure".getBytes();

        SecureMemory memory = new SecureMemory(data.length);
        writeToMemory(memory, data);
        writeToMemoryWithOffset(memory, anotherData);

        readFromMemory(memory);
        readFromMemoryWithOffset(memory, anotherData.length);

        obfuscateMemory(memory);

        deobfuscateMemory(memory);
        deobfuscateMemoryWithOffset(memory, anotherData.length);

        deallocateMemory(memory);
    }

    /**
     * Writes data to the secure memory.
     */
    private static void writeToMemory(SecureMemory memory, byte[] data) {
        memory.write(data);
    }

    /**
     * Writes data to the secure memory with an offset, overwriting the existing data starting from the beginning.
     */
    private static void writeToMemoryWithOffset(SecureMemory memory, byte[] data) {
        memory.write(data, 0);
    }

    /**
     * Reads and prints all data from the secure memory.
     */
    private static void readFromMemory(SecureMemory memory) {
        byte[] readData = memory.read();
        System.out.println(new String(readData)); // prints "Secure World!"
    }

    /**
     * Reads and prints data from the secure memory with a specified offset and length.
     */
    private static void readFromMemoryWithOffset(SecureMemory memory, int length) {
        byte[] offsetRead = memory.read(0, length);
        System.out.println(new String(offsetRead)); // prints "Secure"
    }

    /**
     * Obfuscates the data in memory using XOR with a random key.
     */
    private static void obfuscateMemory(SecureMemory memory) {
        memory.obfuscate();
    }

    /**
     * Deobfuscates and prints all data from the secure memory.
     */
    private static void deobfuscateMemory(SecureMemory memory) {
        var deobfuscated = memory.deobfuscate(String::new);
        System.out.println(deobfuscated); // prints "Secure World!"
    }

    /**
     * Deobfuscates and prints data from the secure memory with a specified offset and length.
     */
    private static void deobfuscateMemoryWithOffset(SecureMemory memory, int length) {
        var partiallyDeobfuscated = memory.deobfuscate(0, length, String::new);
        System.out.println(partiallyDeobfuscated); // prints "Secure"
    }

    /**
     * Deallocates and clears the secure memory.
     */
    private static void deallocateMemory(SecureMemory memory) {
        memory.close();
    }
}
```

### Protecting memory from being read/written/executed
It is enhanced protection for super-sensitive data. You can protect the memory from being read/written/executed or just remove all permissions from the memory, so it will not be accessible at all until you give the permission back.
Under the hood `mprotect` system call is used to change the memory protection flags.

The size of memory allocated should be page aligned. For this, it should be a fully multiple of the page size.
E.g. `pageSize(), pageSize() * 2, pageSize() * 3, ...`
```java
public class MemoryProtectExample {
    public static void main(String[] args) {
        byte[] data = "Hello, World!".getBytes();

        SecureMemory memory = new SecureMemory(pageSize());
        memory.write(data);

        // We block any read or write with execute operation.
        memory.protect(Access.NONE);

        // After this, only read operation will be successful
        memory.protect(Access.READ);

        // After this, only write operation will be successful
        memory.protect(Access.WRITE);

        // After this, any read or write operation will be successful
        // We combine access flags with Bitwise OR operator
        memory.protect(Access.READ | Access.WRITE);

        // When we release the memory, it no write access is set,
        // it wil automatically 
        memory.close();
    }

    /**
     * Size of allocated memory should be page aligned.
     * For this, it should be a fully multiple of the page size.
     * <p>
     * E.g. pageSize(), pageSize() * 2, pageSize() * 3, ...
     */
    private static int pageSize() {
        return MemoryUtil.pageSize();
    }
}
```

### Use cases

#### Secure Application Data Handling
Applications dealing with sensitive information, such as personal user data, payment information, or credentials, can use this library to ensure that such data remains encrypted and protected in memory. This reduces the risk of data leaks through memory dumps or side-channel attacks.

#### Cryptographic Key Management
Cryptographic applications and services, which need to handle encryption keys securely, can benefit from this library. It ensures that keys are kept in protected memory regions, shielded from unauthorized access or exposure, even if the rest of the system is compromised.

#### Password Management Tools
Password managers, which store and manage a large number of user credentials, can utilize this library to safeguard passwords in memory. This ensures that passwords are not exposed in plain text, providing an additional layer of security against memory scraping malware.

#### Financial and Banking Systems
Systems that handle financial transactions, banking details, and sensitive customer data can leverage this library to enhance security. By securing data in memory, these systems can protect against data breaches and ensure regulatory compliance.

#### High-security Environments
In high-security contexts, where data integrity and confidentiality are paramount, this library allows to ensure that all data handled in memory is protected against a wide range possible of vulnerabilities and attacks.


## License
```
MIT License

Copyright (c) 2024 exploit.org

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```