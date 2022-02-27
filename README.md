# DigitalCircuits writeup
CTF逆向二进制题目:数字电路解题技巧

# uncompyle6 version 3.8.0
# Python bytecode 3.7.0 (3394)
# Decompiled from: Python 3.8.8 (default, Apr 13 2021, 12:59:45) 
# [Clang 10.0.0 ]
# Embedded file name: DigitalCircuits.py
# Compiled at: 1995-09-28 00:18:56
# Size of source mod 2**32: 257 bytes
import time
#f1算法:输入a,b同时为1则返回1,异同则返回0(重点不是赋值,而是算法最后返回的值为一个0或者1)
def f1(a, b):
    if a == '1':#[如果a为1则执行:如果b为1则返回1]否则为0
        if b == '1':
            return '1'
    return '0'

#算法f2:a,b同时为0,则返回1,异同则返回0
def f2(a, b):
    if a == '0':
        if b == '0':
            return '0'
    return '1'

#简单的非语句f3,取反(1返回0,为0返回1)
def f3(a):
    if a == '1':
        return '0'
    if a == '0':
        return '1'

#算法f4:返回值是a=(同为1,异为0(a,b取反)),
#             b=(同为1,异为0(a取非,b)).
def f4(a, b):
    return f2(f1(a, f3(b)), f1(f3(a), b))


def f5(x, y, z):
    s = f4(f4(x, y), z)
    c = f2(f1(x, y), f1(z, f2(x, y)))
    return (s, c)


def f6(a, b):
    ans = ''#为一个空字符串
    z = '0'#字符串0
    a = a[::-1]#a从后向前取倒数
    b = b[::-1]#b从后向前取倒数
    for i in range(32):#i赋值从0,1,2,3,,,31
        #ans的值为f5算法的第一位
        ans += f5(a[i], b[i], z)[0]
        #z的值为f5算法的第二位
        z = f5(a[i], b[i], z)[1]
        
    #返回ans的值:从后往前取倒数
    return ans[::-1]

#算法f7:返回a字符串的第n位到结束+n个'0'
def f7(a, n):
    return a[n:] + '0' * n#类型隐式转换

#算法f8:返回除了最后n个数,前面的全部取,即n个'0'+返回数
def f8(a, n):
    return n * '0' + a[:-n]#类型隐式转换

#算法f9:
def f9(a, b):
    ans = ''#定义ans为一个空字符串
    for i in range(32):#i循环从0,1,2,3,,,31
        ans += f4(a[i], b[i])#ans的值为字符串

    return ans


def f10(v0, v1, k0, k1, k2, k3):
    s = '00000000000000000000000000000000'#赋值s为32位0字符串
    d = '10011110001101110111100110111001'#赋值d为1001111000,1101110111,100110111001
    for i in range(32):
        #赋值i=0,1,2,3,4,,,,,31
        s = f6(s, d)#将s,d传入函数f6中:
        v0 = f6(v0, f9(f9(f6(f7(v1, 4), k0), f6(v1, s)), f6(f8(v1, 5), k1)))
        v1 = f6(v1, f9(f9(f6(f7(v0, 4), k2), f6(v0, s)), f6(f8(v0, 5), k3)))

    return v0 + v1


k0 = '0100010001000101'.zfill(32)
k1 = '0100000101000100'.zfill(32)
k2 = '0100001001000101'.zfill(32)
k3 = '0100010101000110'.zfill(32)
flag = 'SUSCTF{xxxxxxxxxxxxxxxxxxxxxxxx}'
if flag[0:7] != 'SUSCTF{' or flag[(-1)] != '}':#flag的1-7位不等于'SUSCTF{'或者左后一位}不等于},那么就报错
    print('Error!!!The formate of flag is SUSCTF{XXX}')
    time.sleep(5)
    exit(0)

flagstr = flag[7:-1]#flagstr赋值flag的大括号内的值
if len(flagstr) != 24:#如果flagstr的字符长度不等于24
    print('Error!!!The length of flag 24')#报错
    time.sleep(5)
    exit(0)
else:
    res = ''#赋值res为空字符串
    for i in range(0, len(flagstr), 8):#0到24,步长为8(0,8,16)
        v0 = flagstr[i:i + 4]#i为0时v0=flagstr第0到4个字符串位置
        #v0={bin(返回整数的二进制形式)「ord(返回单字符字符串的Unicode代码点。)flagstr[字符串第一位]」[去掉0b两字符串后]}.zfill(使用左边填充0共8位)
        #依次i=0,1,2,3
        v0 = bin(ord(flagstr[i]))[2:].zfill(8) + bin(ord(flagstr[(i + 1)]))[2:].zfill(8) + bin(ord(flagstr[(i + 2)]))[2:].zfill(8) + bin(ord(flagstr[(i + 3)]))[2:].zfill(8)
        #v1的值依次为i=4,5,6,7时的值
        v1 = bin(ord(flagstr[(i + 4)]))[2:].zfill(8) + bin(ord(flagstr[(i + 5)]))[2:].zfill(8) + bin(ord(flagstr[(i + 6)]))[2:].zfill(8) + bin(ord(flagstr[(i + 7)]))[2:].zfill(8)
        #res 为f10函数过滤若干值后自加的到
        res += f10(v0, v1, k0, k1, k2, k3)
#res=res-f10(v0,v1,k0,k1,k2,k3)        
    if res == '001111101000100101000111110010111100110010010100010001100011100100110001001101011000001110001000001110110000101101101000100100111101101001100010011100110110000100111011001011100110010000100111':
        print('True')
    else:
        print(i,v0, v1, k0, k1, k2, k3,res,flagstr,flag,len(flagstr))
time.sleep(5)
# okay decompiling DigitalCircuits.pyc







