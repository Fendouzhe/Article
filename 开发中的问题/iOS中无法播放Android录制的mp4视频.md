公司最近在做一个项目，其中有视频录制功能。 
当我们Android端和IOS端都做完后，iOS的同事问我为什么我们Android端录制的视频，他们播放不了。但是iOS录制的视频在Android却能正常播放。 
经过测试在Android端录制的视频，使用iOS的其他应用也无法打开（微信），并且在浏览器中可以播放。 
当时考虑到可能是Android端录制时候视频或者音频编码的问题。后来发现确实是Android端录制时音频编码的问题，之前Android使用的是AMR_NB格式后来换位AAC格式录制后即可解决问题。
```
       mediaRecorder.setAudioEncoder(AudioEncoder.AAC);//音频格式  
       mediaRecorder.setAudioEncoder(AudioEncoder.AMR_NB);//音频格式
```
