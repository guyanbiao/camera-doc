# 功能描述
## Linux App(4g)
1. 录制视频

  ```
  车辆开始运行，只要开始监测到有sd卡，前后摄像头就开始录制视频。
  视频编码格式h264，分辨率为 720p, 25fps，默认切片大小10s。
  音频编码格式为 aac，44100 Hz。空间满了之后循环覆盖旧的文件。
  车辆停止运行结束录制。
  ```
2. 开机之后跟服务器对时

  ```
  GET http://${domain}/sync_time

  response
  {time: "2017-03-08 09:09:00"}
  ```
  
3. 实时将生成的视频文件目录传到服务器

  ```
  文件格式: “${设备ID}_${摄像头位置}YY_MM_DD_HHMMSS”， 例如"5A38748A_front_17_03_18_050302", 表示设备 5A38748A 的前摄像头录制的2017-03-18 05:03:02 视频，摄像头位置: front -> 前，back -> 后

  "设备ID"通过 usb 在开机的时候从 t-box 中读取

  POST http://${domain}/videos

  body {file_name: "5A38748A_front_17_03_18_050302", duration: 10}
```

4. 跟服务器保持长链接，接受服务器指令, 上传指定文件名的视频片段到服务器, 以下为示例代码

  ```ruby
  client = TcpClient.conect_to "http://${domain}/cable"

  #{type: "upload_video", video_files: ["5A38748A_front_17_03_18_050302"]}

  client.on("revice_message") do |msg|
      if msg.type == "upload_video"
        videos = find_by_video_ids msg.video_files
        upload_videos videos
      end 
  end 

  def upload_videos(veidos)
      veidos.each do |v|
        post "http://#{domain}/upload_videos/#{v.file_name}", {body: v.body}
      end
  end 
  ```

5. 接受服务器指令, 进行远程升级

  ```ruby
  client = TcpClient.conect_to "http://${domain}/cable"

  #{type: "upgrade_system", version: "1.3.9", url: "http://domain/firmwares/id/download"}

  client.on("revice_message") do |msg|
      if msg.type == "upgrade_system"
        download_and_update_system(msg.url)
      end 
  end 
  ```

6. 远程设置参数
  * 设置录制切片时长，下一个录制片段采用新的设置
  
    ```
    {type: "set_device", command_type: "slice_duration", value: 10}
    ```

  
## 移动App(wifi)

1. 通过wifi连接到记录仪
2. 读取视频列表，要求带有预览图
3. 下载单个视频到本地
3. 播放单个的视频
4. 删除单个视频
5. 单独播放前摄像头和后摄像头的流，要求延时控制在0.3秒以内，分辨率最低要求为640*480
6. 通过wifi来设置记录仪的参数
  * 修改wifi的密码和名称
  * 设置是否录制声音
  * 显示 SD 卡容量，可以格式化
  * 显示软件版本号
  * 恢复出厂设置
