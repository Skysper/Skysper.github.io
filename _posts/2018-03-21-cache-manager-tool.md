---
layout: post
tId: 1803001
title: "CacheManager�������Դ����"
date: 2018-03-21 22:53:00 +0800
categories: redis,�ܹ�,cache
codelang: html
desc: "���ʹ��cropper�������Ŀ�е�ͼƬ�ϴ����ܣ�����ǿ����ͼƬ�ϴ������һ������"
---

> ������Ҫ�����ҵĿ�Դ��ĿCacheManager��ʹ��˵��������
> https://github.com/Skysper/CacheManager

CacheManager����Э������������ĿӦ����ʹ�õ�Redis��Memcache�����ֵ�ԡ�

Ŀǰ�Ѿ�ʵ���˶�Redis��֧�֣�֧�ֵ��������Ͱ���String��List��Set��SortedSet��Hash�������޸ġ�ɾ����ֵ�����ù���ʱ��ȡ�

### 1. ����App��Ϣ
![����App](//img-blog.csdn.net/20180321221325471?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

����appӦ�ã����������ַ�����֧��Redis��Ⱥ

### 2. ���App��ʹ�õ��ļ�ֵ
![Ӧ��ʹ�û�����](//img-blog.csdn.net/20180321221302912?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


##### ��ֵ����
����Keyֵ���ƴ������԰������ַ�ʽ:

- ����Keyֵ - ֱ�Ӷ�Ӧ������������ض���Key����: `sys:cache:maxTTL` ��
- ����Keyֵ - ����Keyֵ��һ�Զ��ӳ�䣬��Key����һ��Keyֵ���У�������֧������ģʽ
1. ��ֵ���ͣ��Զ�+1��-1�������� `sys:cache:news:{1-10000}`
2. �ַ�����, ��֧��Char���ַ���ʽ���� `sys:cache:cluster{A-Z}`

����ģʽ���Ի��ʹ�ã��� `sys:cache:cluster{A-Z}:news{1-1000}`,���ѯ��Key���б�Ϊ:

```
sys:cache:clusterA:news1
system.cache:clusterA:news2
...
system.cache:clusterA:news1000
system.cache:clusterB:news1
...
cache:clusterZ:news1000
```

*ע������Keyֵģʽ���ʺ���Ե���Id����������ֵ���ַ����������������й���*

### 3.��ѯ����

�趨�����ֵ�͹���ʱ��

![����TTL](//img-blog.csdn.net/20180321222914489?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

�鿴�б�
![�������б�](//img-blog.csdn.net/20180321223210160?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

����б����������ֵ�����Լ���ҳҳ�����ɵ�ǰҳ�ľ���Keyֵ���г���Keyֵ����һ����ʵ������Redis�����У��б���ǰ����ļ�ֵ���ͷֱ�ΪString��List��Set(Sorted Set)��Hash

![Hash�ļ�ֵ����](//img-blog.csdn.net/20180321223122806?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3dwZkxvdmU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### ֵValue���ù���
- String 

```
�ҿ���������������
```

- List��Set��Sorted Set

```
value1
value2
value3
```

- Hash

```
key1$$value1
key2$$value2
key3$$value3
```
