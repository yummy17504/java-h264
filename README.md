# java-h264
常规.264文件解帧
参考自https://github.com/twilightdema/h264j 

在main函数中可修改输入的.264文件路径
new H264Player(new String[]{"H:\\ideaprojectTest\\h264j\\h264\\h264j\\sample_clips\\slamtv10.264"})

解出的每一帧位置可修改savePic函数中的path变量：
String path = "H:/test";

解出的每帧的命名：  newName = path + "/" + oldName + ".jpg";

下一步将进行特殊.264文件头文件中文本信息的提取。
