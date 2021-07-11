# elasticsearch-painless

## About

This project contains painless script so that can be useful when work with Elasticsearch 

## What is Elasticsearch painless language?

Elasticsearch supports a variety of [script language](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)
allowing users can use Elasticsearch in variety flexible way:

| Language | Required plugin | Purpose | 
| -------- | ----------      | -----   | 
| Painless | Build-in        |  Purpose-built for Elasticsearch| 
| Expression | Build-in |   Fast custom ranking and sorting | 
| Java | You must write it | Expert API | 

Painless is build-in script language allowing us can search and analyze data in master manner

## Painless scripts

Because Painless is based on Java programming language, so to write painless script, we should to 
convert Java method to corresponding Painless script. So in each of below script, we have two kind of file:
* Java file: Contain Java method
* Painless file: Is text file contain Painless script corresponding to Java method

### MD5 hash

MD5 is very popular hash digest that can be used in variety of purposes, for example:
* Check data integrity
* A/B testing

#### Painless script:
```
String expectResult = params['expected'];
        char[] val = params['message'].toCharArray();
        byte[] bytes;
        int dp = 0;
        byte[] dst = new byte[val.length << 1];
        for (int sp = 0; sp < val.length; sp++) {
            byte c = (byte) val[sp];
            if (c < 0) {
                dst[dp++] = (byte) (0xc0 | ((c & 255) >> 6));
                dst[dp++] = (byte) (0x80 | (c & 63));
            } else {
                dst[dp++] = c;
            }
        }
        if (dp == dst.length) bytes = dst;
        else {
            bytes = new byte[dp];
            for (int i = 0; i < dp; i++) {
                bytes[i] = dst[i];
            }
        }
        int INIT_A = 0x67452301;
        int INIT_B = (int) 0xEFCDAB89L;
        int INIT_C = (int) 0x98BADCFEL;
        int INIT_D = 0x10325476;
        int[] SHIFT_AMTS = new int[]{7, 12, 17, 22, 5, 9, 14, 20, 4, 11, 16, 23, 6, 10, 15, 21};
        int[] TABLE_T = new int[64];
        for (int i = 0; i < 64; i++) TABLE_T[i] = (int) (long) ((1L << 32) * Math.abs(Math.sin(i + 1)));
        int messageLenBytes = bytes.length;
        int numBlocks = ((messageLenBytes + 8) >>> 6) + 1;
        int totalLen = numBlocks << 6;
        byte[] paddingBytes = new byte[totalLen - messageLenBytes];
        paddingBytes[0] = (byte) 0x80;
        long messageLenBits = (long) messageLenBytes << 3;
        for (int i = 0; i < 8; i++) {
            paddingBytes[paddingBytes.length - 8 + i] = (byte) messageLenBits;
            messageLenBits >>>= 8;
        }
        int a = INIT_A;
        int b = INIT_B;
        int c = INIT_C;
        int d = INIT_D;
        int[] buffer = new int[16];
        for (int i = 0; i < numBlocks; i++) {
            int index = i << 6;
            for (int j = 0; j < 64; j++) {
                buffer[j >>> 2] = ((int) ((index < messageLenBytes) ? bytes[index] : paddingBytes[index - messageLenBytes]) << 24) | (buffer[j >>> 2] >>> 8);
                index++;
            }
            int originalA = a;
            int originalB = b;
            int originalC = c;
            int originalD = d;
            for (int j = 0; j < 64; j++) {
                int div16 = j >>> 4;
                int f = 0;
                int bufferIndex = j;
                if (div16 == 0) {
                    f = (b & c) | (~b & d);
                } else if (div16 == 1) {
                    f = (b & d) | (c & ~d);
                    bufferIndex = (bufferIndex * 5 + 1) & 15;
                } else if (div16 == 2) {
                    f = b ^ c ^ d;
                    bufferIndex = (bufferIndex * 3 + 5) & 15;
                } else if (div16 == 3) {
                    f = c ^ (b | ~d);
                    bufferIndex = (bufferIndex * 7) & 15;
                }
                int temp = b + Integer.rotateLeft(a + f + buffer[bufferIndex] + TABLE_T[j], SHIFT_AMTS[(div16 << 2) | (j & 3)]);
                a = d;
                d = c;
                c = b;
                b = temp;
            }
            a += originalA;
            b += originalB;
            c += originalC;
            d += originalD;
        }
        byte[] md5 = new byte[16];
        int count = 0;
        for (int i = 0; i < 4; i++) {
            int n = (i == 0) ? a : ((i == 1) ? b : ((i == 2) ? c : d));
            for (int j = 0; j < 4; j++) {
                md5[count++] = (byte) n;
                n >>>= 8;
            }
        }
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < md5.length; i++) {
            int k = md5[i] & 255;
            int div = k / 16;
            int mod = k % 16;
            if (div < 10) {
                sb.append(div);
            } else {
                sb.append((char) (65 + div - 10));
            }
            if (mod < 10) {
                sb.append(mod); 
            } else {
                sb.append((char) (65 + mod - 10));
            }
        }
        return sb.toString().toUpperCase().equals(expectResult);
```

#### How to test this script:
For example, we have:
* message: "This is painless"
* With md5 hash: "3B3EAF9D8F558453E735F3421954E440"

We execute below query to Elasticsearch:
``` 
GET _scripts/painless/_execute
{
   "script": {
    "lang": "painless",
    "source": """String expectResult = params['expected'];
        char[] val = params['message'].toCharArray();
        byte[] bytes;
        int dp = 0;
        byte[] dst = new byte[val.length << 1];
        for (int sp = 0; sp < val.length; sp++) {
            byte c = (byte) val[sp];
            if (c < 0) {
                dst[dp++] = (byte) (0xc0 | ((c & 255) >> 6));
                dst[dp++] = (byte) (0x80 | (c & 63));
            } else {
                dst[dp++] = c;
            }
        }
        if (dp == dst.length) bytes = dst;
        else {
            bytes = new byte[dp];
            for (int i = 0; i < dp; i++) {
                bytes[i] = dst[i];
            }
        }
        int INIT_A = 0x67452301;
        int INIT_B = (int) 0xEFCDAB89L;
        int INIT_C = (int) 0x98BADCFEL;
        int INIT_D = 0x10325476;
        int[] SHIFT_AMTS = new int[]{7, 12, 17, 22, 5, 9, 14, 20, 4, 11, 16, 23, 6, 10, 15, 21};
        int[] TABLE_T = new int[64];
        for (int i = 0; i < 64; i++) TABLE_T[i] = (int) (long) ((1L << 32) * Math.abs(Math.sin(i + 1)));
        int messageLenBytes = bytes.length;
        int numBlocks = ((messageLenBytes + 8) >>> 6) + 1;
        int totalLen = numBlocks << 6;
        byte[] paddingBytes = new byte[totalLen - messageLenBytes];
        paddingBytes[0] = (byte) 0x80;
        long messageLenBits = (long) messageLenBytes << 3;
        for (int i = 0; i < 8; i++) {
            paddingBytes[paddingBytes.length - 8 + i] = (byte) messageLenBits;
            messageLenBits >>>= 8;
        }
        int a = INIT_A;
        int b = INIT_B;
        int c = INIT_C;
        int d = INIT_D;
        int[] buffer = new int[16];
        for (int i = 0; i < numBlocks; i++) {
            int index = i << 6;
            for (int j = 0; j < 64; j++) {
                buffer[j >>> 2] = ((int) ((index < messageLenBytes) ? bytes[index] : paddingBytes[index - messageLenBytes]) << 24) | (buffer[j >>> 2] >>> 8);
                index++;
            }
            int originalA = a;
            int originalB = b;
            int originalC = c;
            int originalD = d;
            for (int j = 0; j < 64; j++) {
                int div16 = j >>> 4;
                int f = 0;
                int bufferIndex = j;
                if (div16 == 0) {
                    f = (b & c) | (~b & d);
                } else if (div16 == 1) {
                    f = (b & d) | (c & ~d);
                    bufferIndex = (bufferIndex * 5 + 1) & 15;
                } else if (div16 == 2) {
                    f = b ^ c ^ d;
                    bufferIndex = (bufferIndex * 3 + 5) & 15;
                } else if (div16 == 3) {
                    f = c ^ (b | ~d);
                    bufferIndex = (bufferIndex * 7) & 15;
                }
                int temp = b + Integer.rotateLeft(a + f + buffer[bufferIndex] + TABLE_T[j], SHIFT_AMTS[(div16 << 2) | (j & 3)]);
                a = d;
                d = c;
                c = b;
                b = temp;
            }
            a += originalA;
            b += originalB;
            c += originalC;
            d += originalD;
        }
        byte[] md5 = new byte[16];
        int count = 0;
        for (int i = 0; i < 4; i++) {
            int n = (i == 0) ? a : ((i == 1) ? b : ((i == 2) ? c : d));
            for (int j = 0; j < 4; j++) {
                md5[count++] = (byte) n;
                n >>>= 8;
            }
        }
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < md5.length; i++) {
            int k = md5[i] & 255;
            int div = k / 16;
            int mod = k % 16;
            if (div < 10) {
                sb.append(div);
            } else {
                sb.append((char) (65 + div - 10));
            }
            if (mod < 10) {
                sb.append(mod); 
            } else {
                sb.append((char) (65 + mod - 10));
            }
        }
        return sb.toString().toUpperCase().equals(expectResult);"""
        ,
  "params": {
      "message": "This is painless",
      "expected": "3B3EAF9D8F558453E735F3421954E440"
    }
   }
}
```

We got the result:
``` 
{
  "result" : "true"
}
```
-> script work correctly

-> We can customize above script to compare the hash value with the values we wants 