---
layout: post
title: OpenSSL中添加自定义加密算法
category: Linux
tags: Linux
keywords: openssl,加密
description: 
---

## 引言
---
>    本文以添加自定义算法EVP_ssf33为例，介绍在OpenSSL中添加自定义加密算法的方法。

## 步骤
---
    1、修改crypto/object/objects.txt，注册算法OID，如下：
    
        rsadsi 3 255    : SSF33     : ssf33
    
    2、进入目录：crypto/object/，执行如下命令，生成算法的声明
    
        perl objects.pl objects.txt obj_mac.num obj_mac.h
    
    3、在crypto/evp/下添加e_ssf33.c，内容如下：
````c   
    #include <stdio.h>
    #include "cryptlib.h"
    #ifndef OPENSSL_NO_RC4
        #include <openssl/evp.h>
        #include <openssl/objects.h>
        #include <openssl/rc4.h>
        
        /* FIXME: surely this is available elsewhere? */
        #define EVP_SSF33_KEY_SIZE      16
        
        typedef struct
        {
            RC4_KEY ks; /* working key */
        } EVP_SSF33_KEY;
        
        #define data(ctx) ((EVP_SSF33_KEY *)(ctx)->cipher_data)
        
        static int ssf33_init_key(EVP_CIPHER_CTX *ctx, const unsigned char *key, const unsigned char *iv,int enc);
        
        static int ssf33_cipher(EVP_CIPHER_CTX *ctx, unsigned char *out, const unsigned char *in, unsigned int inl);
        
        static const EVP_CIPHER ssf33_evp_cipher=
        {
            NID_ssf33,
            1,
            EVP_SSF33_KEY_SIZE,
            0,
            EVP_CIPH_VARIABLE_LENGTH,
            ssf33_init_key,
            ssf33_cipher,
            NULL,
            sizeof(EVP_SSF33_KEY),
            NULL,
            NULL,
            NULL,
            NULL
        };
        
        const EVP_CIPHER *EVP_ssf33(void)
        {
            return(&ssf33_evp_cipher);
        }
        
        static int ssf33_init_key(EVP_CIPHER_CTX *ctx, const unsigned char *key, const unsigned char *iv, int enc)
        {
            RC4_set_key(&data(ctx)->ks,EVP_CIPHER_CTX_key_length(ctx), key);
        
            return 1;
        }
        
        static int ssf33_cipher(EVP_CIPHER_CTX *ctx, unsigned char *out, const unsigned char *in, unsigned int inl)
        {
            RC4(&data(ctx)->ks,inl,in,out);
        
            return 1;
        }
    
    #endif
````    
    4、修改crypto/evp/evp.h，添加对算法的声明，如下
        
        const EVP_CIPHER *EVP_ssf33(void);
        
    5、修改crypto/evp/c_allc.c，在OpenSSL_add_all_ciphers函数中使用EVP_add_cipher注册加密函数，如下
        
        EVP_add_cipher(EVP_ssf33());
        
    6、修改crypto/evp/Makefile，如下
    
   ![]({{site.url}}/assets/uploads/openssl-custom-algorithms-1.png)
    
    7、完成

感谢收看，如果对大家有帮助，请[github上follow和star](https://github.com/tongshunmin)，本文发布在[佟顺民的技术博客](http://blog.mineki.cn/)，转载请注明出处

##  参考
使用 DEF 文件从 DLL 导出
[Module-Definition (.Def) Files](http://my.oschina.net/hondfy/blog/165675)


