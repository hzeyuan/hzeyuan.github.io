---
layout: post
title: "Twitter-snowflake����Ψһ��id�㷨"
subtitle: 'snowflake����Ψһ��id�㷨'
author: "Hzy"
header-style: text
tags:
  - �㷨

---

### �����춼�����������Ψһ��iD�����⣬������һЩ����

#### 1.�󲿷����¶���������¼��ַ�����

##### 1.1 �������ݿ�����ID

>�ŵ㣺

* ʵ�ּ򵥣�ID����

>ȱ�㣺

* ��Ҫ���������,��������Ҫ��ID��Ҫ���м��ɺ��и�ʱ����α�֤ID����ͻ��
* ���磺 �¾�ϵͳ�����ڵ�ʱ��
* ���磺 ��Ҫ����ָ��ID���м�¼��ʱ��

##### 1.2 UUID

>�ŵ㣺

* �������ɣ����ɼ򵥣����ܺã�û�и߿��÷���

> ȱ�㣺

* ���ȹ������洢���࣬�����򲻿ɶ�����ѯЧ�ʵ�

##### 1.3 Redis����ID,�������û�Թ�����Ҫ��redis��ʱ������

> �ŵ㣺

* �����������ݿ⣬���㣬�������������ݿ⣻���򣬶Է�ҳ������Ҫ����Ľ�����а�����
* ���ڷֲ�ʽ��˵�����Գ�ʼ��ÿ̨redis�ĳ�ʼֵ��������Ȼ���ղ���������

```
A��1, 6, 11, 16, 21
B��2, 7, 12, 17, 22
C��3, 8, 13, 18, 23
D��4, 9, 14, 19, 24
E��5, 10, 15, 20, 25
```

> ȱ�㣺
* ����ѹ������redis��ϵͳ����Ҫ������������


##### 1.4 snowflake�㷨�������ص㡣

![](/img/snowflake.png)

* 1bit:���ű���
* 41bit: ʱ�����2��41�η�Ϊ��2199023255552�����赱ǰʱ��Ϊ2020-04-18 01:38:18��תΪʱ���1587145098000��   1587145098000+2199023255552=3786168353552��3786168353552����2089-12-23 17:25:53������ʹ��69�ꡣ
* 10bit:��¼����id��2��10�η���1024̨���������Ը��ݵ���͹��������ڽ���ϸ�֡�

> ��ʵ����iD��ʱ��������Ը����Լ��������壬41λ������Ϊ�պÿ�����������BIGINT�����С�

##### ��������python�е�ʵ��

> 1.��������ı���,�ֲ����򣬻����룬���е�bit

```
region_id_bits = 2 #�����ֲ�����������bit -> 2**2=4
worker_id_bits = 10 # #���������bit ->2**10=1024
sequence_bits = 11 # ���з����bit ->2**11=2028
```

> 2.�������ֵ������ȡ�����ֵ

```
# ��ȡ�����ֵ��λ����+1
MAX_REGION_ID = -1 ^ (-1 << region_id_bits)
MAX_WORKER_ID = -1 ^ (-1 << worker_id_bits)
SEQUENCE_MASK = -1 ^ (-1 << sequence_bits)
```


> 3.����һ��Ԫ�꣬��Twitter����1288834974657,ʱ�����-11��ʼ�����к�Ϊ0

```
self.twepoch = 1288834974657
self.last_timestamp = -1
self.sequence = 0
```

> 4.����id�ĺ���
* �ж�ʱ���Ƿ�ͬ�����ֲ�ʽ�Ļ���ʱ�����һ�£����ǵ�̨������û�����⡣
* �жϵ�ǰ���к��Ƿ񳬳����ֵ������Ѱ���µ�ʱ��������������к�+1��
* ����ʱ���������id,����id�����кţ�����ƫ��Ȼ��ƴ�ӡ�

```
# ������ܻ�㿴��̫������ʵ���������ƶ������(<<)���0��
# ��ʱ�������64λ
# ������id������idͬ������
# ���ʹ�û����㽫���Ǽ��������γ�һ��64bit���ȵ�Ψһid.
return (
    (timestamp - self.twepoch) << Snowflake.TIMESTAMP_LEFT_SHIFT 
    |(padding_num << Snowflake.REGION_ID_SHIFT) 
    |(self.worker_id << Snowflake.WORKER_ID_SHIFT) 
    |self.sequence )
```



### �������ɵĴ��룺
```
class Snowflake(object):
    region_id_bits = 2
    worker_id_bits = 10
    sequence_bits = 11

    # ��ȡ�����ֵ��λ����+1
    MAX_REGION_ID = -1 ^ (-1 << region_id_bits)
    MAX_WORKER_ID = -1 ^ (-1 << worker_id_bits)
    SEQUENCE_MASK = -1 ^ (-1 << sequence_bits)

    # ��λƫ�Ƽ��㣬����λ��������֮����������
    WORKER_ID_SHIFT = sequence_bits
    REGION_ID_SHIFT = sequence_bits + worker_id_bits
    TIMESTAMP_LEFT_SHIFT = (sequence_bits + worker_id_bits + region_id_bits)

    def __init__(self, worker_id, region_id=0):
        self.twepoch = 1288834974657
        self.last_timestamp = -1
        self.sequence = 0

        assert 0 <= worker_id <= Snowflake.MAX_WORKER_ID
        assert 0 <= region_id <= Snowflake.MAX_REGION_ID

        self.worker_id = worker_id
        self.region_id = region_id

        self.lock = threading.Lock()

    def generate(self, bus_id=None):
        return self.next_id(
            True if bus_id is not None else False,
            bus_id if bus_id is not None else 0
        )

    def next_id(self, is_padding, bus_id):
        with self.lock:
            timestamp = self.get_time()
            padding_num = self.region_id

            if is_padding:
                padding_num = bus_id

            if timestamp < self.last_timestamp:
                try:
                    raise ValueError(
                        'Clock moved backwards. Refusing to'
                        'generate id for {0} milliseconds.'.format(
                            self.last_timestamp - timestamp
                        )
                    )
                except ValueError:
                    print(sys.exc_info()[2])

            if timestamp == self.last_timestamp:
                self.sequence = (self.sequence + 1) & Snowflake.SEQUENCE_MASK
                if self.sequence == 0:
                    timestamp = self.tail_next_millis(self.last_timestamp)
            else:
                self.sequence = random.randint(0, 9)

            self.last_timestamp = timestamp

            print(timestamp - self.twepoch << Snowflake.TIMESTAMP_LEFT_SHIFT)
            print(padding_num << Snowflake.REGION_ID_SHIFT)
            print(self.worker_id << Snowflake.WORKER_ID_SHIFT)
            print(self.sequence)
            return (
                    (timestamp - self.twepoch) << Snowflake.TIMESTAMP_LEFT_SHIFT |
                    (padding_num << Snowflake.REGION_ID_SHIFT) |
                    (self.worker_id << Snowflake.WORKER_ID_SHIFT) |
                    self.sequence
            )

    def tail_next_millis(self, last_timestamp):
        timestamp = self.get_time()
        while timestamp <= last_timestamp:
            timestamp = self.get_time()
        return timestamp

    def get_time(self):
        return int(time.time() * 1000)

```