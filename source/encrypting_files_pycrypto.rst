Encrypting files with Pycrypto
##############################

:date: 2014-11-04 20:58
:author: Dejan Noveski
:category: Coding
:tags: python, coding, crypto, encryption, pycrypto


`Pycrypto <https://www.dlitz.net/software/pycrypto/>`_ is the python cryptography toolkit you'll ever need. It supports pretty much everything, it's
rather fast and has an understandable interface and documentation. Cryptography, in itself, is rather complicated and hard to understand and implement,
and so is the documentation on encrypting files with pycrypto (or pretty much any crypto toolkit/library in the world).

Recently, I was tasked with writing a code that will encrypt files to be decrypted on a mobile platform - Android and iOS. Both of their documentations are
lacking in that department. You can easily find a snippet so that you can encrypt and decrypt a file on the same device, but not something that will work
cross-device/cross-platform. That's because some of the details of the encryption algorithms are not mentioned and those are crucial.


ELI5 AES
--------

AES is a block cypher. That means, it encrypts/decrypts in blocks of 128 bits. The key used for the [en|de]cryption is 128, 192 or 256 bits.
So, this means that your data has to be divisible by 128bits and your key must be 128, 192 or 256 bits. Pretty tough to meet these requirements considering
we're dealing with binary/text files that will be some random length? Well, that's where padding comes in.

In order to make the file's length divisible by **128b(16B)** we have to pad it. The most used padding scheme is `PKCS7 <http://en.wikipedia.org/wiki/Padding_%28cryptography%29#PKCS7>`_
Simply append n < 16 bytes to the file with value n to meet the 16B block size. So, the workflow now would be, pad -> encrypt -> decrypt -> unpad.

There's one more thing - IV - Should also be 16B. It's a common practice to have this at random, prepend it to the file on every encryption. We'll assume a static IV now for the sake of
simplicity. The documentations around the net don't mention the IV. Then, the crypto system creates it per device, and you'll be stuck wondering how you can
decrypt on your phone but not your desktop.


Enter AES256/CBC/PKCS7
----------------------

This is the cookie cutter scheme for encryption. It's supported by the mobile OS-es and libraries, it's even the default!

Doing this with openssl is easy:


.. code-block:: bash
    
    # To Encrypt 
    openssl enc -aes-256-cbc -k LyoNIpj8nqg5tcukqmW3kJ7PIbHtfeHE -iv 0000000000000000 -in in_file -out out_file
    
    # To Decrypt
    openssl enc -d -aes-256-cbc -k LyoNIpj8nqg5tcukqmW3kJ7PIbHtfeHE -iv 0000000000000000 -in in_file -out out_file
    

In this example, the key is set to *LyoNIpj8nqg5tcukqmW3kJ7PIbHtfeHE* **(32B/256b)** and the IV is set to 16B of zeroes. Openssl pads with PKCS7 by default.

Now let's see the example with pycrypto. Assuming with have in_file and out_file already opened:

.. code-block:: python
    
    # initialize the encryption
    encryption_key = "LyoNIpj8nqg5tcukqmW3kJ7PIbHtfeHE"
    iv = 16 * '\x00'
    crypt = AES.new(encryption_key, AES.MODE_CBC, iv)
    
    # keep chunk size large for speed but divisible by 16B
    chunksize = 1024
    
    while True:
	chunk = infile.read(chunksize)
        if len(chunk) == 0:
	    # it's an empty chunk. We don't need it.
	    break
        elif len(chunk) == chunksize:
	    # We've read a full encryptable chunk with length divisible by 16B
            out_file.write(crypt.encrypt(chunk))
        else:
            # We've read a chunk that's not divisible by 16B. We PCKS7 pad it.
            # First calculate how many bytes we'll need to pad it
            padding_bytes = 16 - len(chunk) % AES.block_size
            # Next, create the padding sequence
            padding = StringIO.StringIO()
            for _ in xrange(padding_bytes):
		# If we're missing 4 bytes, the padding sequence would be 04 04 04 04 (hex). That's why the formatting.
                padding.write('%02x' % padding_bytes)
            padded_chunk = chunk + binascii.unhexlify(padding.getvalue())
            out_file.write(crypt.encrypt(padded_chunk))

                
That's it. This should be decryptable with openssl, android crypto, ios crypto library and any device. Just need to know the encryption scheme, the IV and the encryption key.
If you really want to make something secure, I suggest you think about making a random IV each time the function is called.
