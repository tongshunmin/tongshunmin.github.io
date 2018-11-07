---
layout: post
title: OpenSSL编程
category: Linux
tags: Linux
keywords: OpenSSL,编程
description: 
---

## 引言

>    OpenSSL是一个功能丰富且自包含的开源安全工具箱。它提供的主要功能有：SSL协议实现(包括SSLv2、SSLv3和TLSv1)、大量软算法(对称/非对称/摘要)、大数运算、非对称算法密钥生成、ASN.1编解码库、证书请求(PKCS10)编解码、数字证书编解码、CRL编解码、OCSP协议、数字证书验证、PKCS7标准实现和PKCS12个人数字证书格式实现等功能。     
     OpenSSL采用C语言作为开发语言，这使得它具有优秀的跨平台性能。OpenSSL支持Linux、UNIX、windows、Mac等平台。OpenSSL目前最新的版本是openssl-1.0.0e.。

## 编译安装
    
1、资源下载

 [http://www.openssl.org/source/old/1.0.1/openssl-1.0.1l.tar.gz](http://www.openssl.org/source/old/1.0.1/openssl-1.0.1l.tar.gz)
 
2、编译安装
 
 [http://www.openssl.org/source/old/1.0.1/openssl-1.0.1l.tar.gz](http://www.openssl.org/source/old/1.0.1/openssl-1.0.1l.tar.gz)

## API文档

 [http://www.openssl.org/docs/crypto/crypto.html](http://www.openssl.org/docs/crypto/crypto.html)
 
 [http://www.openssl.org/docs/ssl/ssl.html](http://www.openssl.org/docs/ssl/ssl.html)

## 编程示例

### 程序1：openssl堆栈示例 

````c++
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <openssl/safestack.h>
    #define sk_Student_new(st) SKM_sk_new(Student, (st))
    #define sk_Student_new_null() SKM_sk_new_null(Student)
    #define sk_Student_free(st) SKM_sk_free(Student, (st))
    #define sk_Student_num(st) SKM_sk_num(Student, (st))
    #define sk_Student_value(st, i) SKM_sk_value(Student, (st), (i))
    #define sk_Student_set(st, i, val) SKM_sk_set(Student, (st), (i), (val))
    #define sk_Student_zero(st) SKM_sk_zero(Student, (st))
    #define sk_Student_push(st, val) SKM_sk_push(Student, (st), (val))
    #define sk_Student_unshift(st, val) SKM_sk_unshift(Student, (st), (val))
    #define sk_Student_find(st, val) SKM_sk_find(Student, (st), (val))
    #define sk_Student_delete(st, i) SKM_sk_delete(Student, (st), (i))
    #define sk_Student_delete_ptr(st, ptr) SKM_sk_delete_ptr(Student, (st), (ptr))
    #define sk_Student_insert(st, val, i) SKM_sk_insert(Student, (st), (val), (i))
    #define sk_Student_set_cmp_func(st, cmp) SKM_sk_set_cmp_func(Student, (st), (cmp))
    #define sk_Student_dup(st) SKM_sk_dup(Student, st)
    #define sk_Student_pop_free(st, free_func) SKM_sk_pop_free(Student, (st), (free_func))
    #define sk_Student_shift(st) SKM_sk_shift(Student, (st))
    #define sk_Student_pop(st) SKM_sk_pop(Student, (st))
    #define sk_Student_sort(st) SKM_sk_sort(Student, (st))
    
    typedef struct Student_st
    {
        char *name;
        int age;
        char *otherInfo;
    } Student;
    
    typedef STACK_OF(Student) Students;
    
    Student *Student_Malloc()
    {
        Student *a=malloc(sizeof(Student));
            a->name=(char *)malloc(sizeof(char)*20);
        strcpy(a->name,"zcp");
        a->otherInfo=(char *)malloc(sizeof(char)*20);     strcpy(a->otherInfo,"no info");
        return a;
    }
    
    void Student_Free(Student *a)
    {
        free(a->name);
        free(a->otherInfo);
        free(a);
    }
    static int Student_cmp(Student *a,Student *b)
    {
        int ret;
        ret=strcmp(a->name,b->name);  /* 只比较关键字 */
        return ret;
    }
    
    int main()
    {
        Students *s,*snew;
        Student *s1,*one,*s2;
        int i,num;
        s=sk_Student_new_null();          /* 新建一个堆栈对象 */
        snew=sk_Student_new(Student_cmp); /* 新建一个堆栈对象 */
        s2=Student_Malloc();
        sk_Student_push(snew,s2);
        i=sk_Student_find(snew,s2);
        s1=Student_Malloc();
        sk_Student_push(s,s1);
        num=sk_Student_num(s);
        for(i=0; i<num; i++)
        {
            one=sk_Student_value(s,i);
            printf("student name : %s\n",one->name);
            printf("sutdent age : %d\n",one->age);
            printf("student otherinfo : %s\n\n\n",one->otherInfo);
        }
           sk_Student_pop_free(s,Student_Free);
        sk_Student_pop_free(snew,Student_Free);
        
        return 0;
    }
````     

####编译

    gcc example1.c -o example1 -L/usr/lib -lssl -lcrypto
    
####运行

   ![]({{site.url}}/assets/uploads/openssl-program-1.png)

###程序2：openssl哈希表示例 
   
````c++
    #include <string.h>
    #include <openssl/lhash.h>
    
    typedef struct Student_st
    {
        char name[20];
        int age;
        char otherInfo[200];
    } Student;
    
    static int Student_cmp(const void *a, const void *b)
    {
        char *namea=((Student *)a)->name;
        char *nameb=((Student *)b)->name;
        return strcmp(namea,nameb);
    }
    
    /* 打印每个值*/
    static void PrintValue(Student *a)
    {
        printf("name :%s\n",a->name);
        printf("age  :%d\n",a->age);
        printf("otherInfo : %s\n",a->otherInfo);
    }
    
    static void PrintValue_arg(Student *a,void *b)
    {
        int flag=0;
    
        flag=*(int *)b;
        printf("用户输入参数为:%d\n",flag);
        printf("name :%s\n",a->name);
        printf("age  :%d\n",a->age);
        printf("otherInfo : %s\n",a->otherInfo);
    }
    
    int main()
    {
        int flag=11;
        _LHASH *h;
        Student s1= {"zcp",28,"hu bei"},
                s2= {"forxy",28,"no info"},
                s3= {"skp",24,"student"},
                s4= {"zhao_zcp",28,"zcp's name"},
                *s5;
        void *data;
    
        /*创建哈希表*/
        h=lh_new(NULL,Student_cmp); 
        if(h==NULL)
        {
            printf("err.\n");
            return -1;
        }
        /*将数据插入哈希表*/
        data=&s1;
        lh_insert(h,data);
        data=&s2;
        lh_insert(h,data);
        data=&s3;
        lh_insert(h,data);
        data=&s4;
        lh_insert(h,data);
        
        /*遍历打印*/
        lh_doall(h,PrintValue);
        lh_doall_arg(h,PrintValue_arg,(void *)(&flag));
        
        /*查找数据*/
        data=lh_retrieve(h,(const void*)"skp");
        if(data==NULL)
        {
            printf("can not find skp!\n");
            lh_free(h);
            return -1;
        }
        else
        {
            s5=data;
            printf("\n\nstudent name : %s\n",s5->name);
            printf("sutdent age  : %d\n",s5->age);
            printf("student otherinfo : %s\n",s5->otherInfo);
            lh_free(h);
        }
        
        getchar();
        return 0;
    }    
```` 

####编译

    gcc example2.c -o example2 -L/usr/lib -lssl -lcrypto
    
####运行

   ![]({{site.url}}/assets/uploads/openssl-program-2.png)
   
###程序3：openssl内存管理示例   

````c++
    #include <openssl/crypto.h>
    #include <openssl/bio.h>
    
    int main()
    {
        char *p;
        BIO *b;
        CRYPTO_malloc_debug_init();
        CRYPTO_set_mem_debug_options(V_CRYPTO_MDEBUG_ALL);
        CRYPTO_mem_ctrl(CRYPTO_MEM_CHECK_ON); /*开户内存记录*/
        
        p=OPENSSL_malloc(4);
        CRYPTO_mem_ctrl(CRYPTO_MEM_CHECK_OFF);/*关闭内存记录*/
        b=BIO_new_file("leak.log","w");
        CRYPTO_mem_ctrl(CRYPTO_MEM_CHECK_ON);
        CRYPTO_mem_leaks(b);    /*将内存泄露输出到FILE中*/
        
        OPENSSL_free(p);
        BIO_free(b);
        return 0;
    }
```` 

####编译

    gcc example3.c -o example3 -L/usr/lib -lssl -lcrypto
    
####运行
   ![]({{site.url}}/assets/uploads/openssl-program-3.png)




感谢收看，如果对大家有帮助，请[github上follow和star](https://github.com/tongshunmin)，本文发布在[佟顺民的技术博客](http://blog.mineki.cn/)，转载请注明出处

##  参考
使用 DEF 文件从 DLL 导出
[Module-Definition (.Def) Files](http://my.oschina.net/hondfy/blog/165675)


