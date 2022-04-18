# java-h264
## 常规.264文件解帧
参考自https://github.com/twilightdema/h264j   

在main函数中可修改输入的.264文件路径
new H264Player(new String[]{"H:\\ideaprojectTest\\h264j\\h264\\h264j\\sample_clips\\slamtv10.264"})  

解出的每一帧位置可修改savePic函数中的path变量：
String path = "H:/test";  

解出的每帧的命名：  newName = path + "/" + oldName + ".jpg";  

下一步将进行特殊.264文件头文件中文本信息的提取。

## h264文件格式介绍
H.264的码流文件分为两层，视频编码层(VCL)和网络提取层(NAL)  
VCL：视频编码层，进行视频编解码  
NAL：网络提取层，采用适当的格式对视频数据进行封装打包

VCL数据即被压缩编码后的视频数据序列，在VCL数据封装到NAL单元中之后，才可以用来传输或存储。

**NAL单元(NALU)**  
NALU由【起始码】【NAL头】【一个不定长编码段(payload)】组成

其中，【起始码】为：0x00 00 01 或 0x00 00 00 01，NAL头由1个字节组成，NAL PayLoad由不定长编码段组成。  
例如一个H.264的NALU为：  
[00 00 00 01 67 42 A0 1E 23 56 0E 2F ….]  
则：【00 00 00 01】是四个字节的开始码  
   【67】是1字节的NAL头  
   【42 A0 ….】是NAL的内容  

【NALU头】由一个字节组成
语法：NALU类型位（5bit）、重要性位（2bit）、禁止位（1bit）
位	0	1~2	3~7
名称	F(forbidden_zero_bit)
禁止位	NRI(nal_ref_idc)重要性标志位	TYPE(nal_unit_type)
NALU类型位
作用	网络发现NAL单元有比特错误时可设置该比特为1，一遍接收方丢掉该单元	标志该NAL单元用于重建时的重要性，值越大越重要，取00~11	1~23位表示单个NAL包，24~31需要分包或者组合发送。
下面是NALU类型取值含义：
0：没有定义
1~23：NAL单元，单个NAL单元包           24：STAP-A单一时间的组合包
1：不分区，非IDR图像的片                25：STAP-B 单一时间的组合包
2：片分区A                               26：MTAP16多个时间的组合包
3：片分区B                               27：MTAP24多个时间的组合包
4：片分区C                               28：FU-A分片的单元
5：IDR图像中的片                         29：FU-B分片的单元
6：补充增强信息单元（SEI）                30~31：没有定义
7：SPS
8：PPS
9：序列结束
10：序列结束
11：码流结束
12：填充
12~23：保留

假设NALU头如下所示：
【67】==》【0110 0111】
则它代表：
禁止位为【0】  
NAL重要性为【11】，即3，表示最重要  
NALU类型为【0 0111】，即7，从上面的表格可以看出，这个NALU是SPS  

【NALU payload】
NALU的主体涉及到三个重要的名词，分别为EBSP、RBSP和SODB。其中EBSP完全等价于NALU主体，而且它们三个的结构关系为：EBSP包含RBSP，RBSP包含SODB。  
1. SODB: String Of Data Bits 原始数据比特流，就是最原始的编码/压缩得到的数据  
2. RBSP: Raw Byte Sequence Payload，又称原始字节序列载荷。  
和SODB关系如下：  
RBSP = SODB + RBSP Trailing Bits（RBSP尾部补齐字节）  
引入RBSP Trailing Bits做8位字节补齐。  
3. EBSP: Encapsulated Byte Sequence Payload: 扩展字节序列载荷  
如果RBSP中也包括了StartCode（0x000001或0x00000001）怎么办呢？所以，就有了防止竞争字节（0x03）. 编码时，扫描RBSP，如果遇到连续两个0x00字节，就在后面添加防止竞争字节（0x03）；解码时，同样扫描EBSP，进行逆向操作即可。  
当编码器编完一个NALU时，
应该检测NALU内部是否出现了如下左侧的数据，如果检测到它们的存在，
编码器就在最后一个字节前，插入一个新字节“0x03”：  
0x000000 --> 0x00000300  
0x000001 --> 0x00000301  
0x000002 --> 0x00000302  
0x000003 --> 0x00000303  
注：
NALU的头是0x000001或0x00000001，都按上面的方式处理，  
对于0x000001就是在第2个字节后插入0x03，  
对于0x00000001就是在第3个字节后插入0x03。  
当解码器解码时，将0x03去掉即可，也称为脱壳操作。  
注：在H264的文档中，并没有EBSP这一名词出现，但是在H264的官方参考软件JM里，却使用了EBSP。  
EBSP = RBSP + “0x03”
