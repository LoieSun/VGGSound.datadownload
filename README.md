# VGGSound.datadownload
download the VGGSound data from YouTube by using Colab

## Read VGGSound file(vidoe id, label, ...)<br/><br/><br/><br/>

## Open Colab
 drive to Google disk, run the following code
 ````python
  from google.colab import drive
  drive.mount('/content/drive', force_remount=True
  %cd /content/drive/My Drive/
  !mkdir Download
  %cd /content/drive/My Drive/Download
 ````
 <br/><br/>
 
 download ffmpeg, run the following code
 ````python
  % cd /content/sample_data
  ! wget https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz
  ! tar -xf ffmpeg-release-amd64-static.tar.xz
 ````
 <br/><br/>
 
 pip packages, run code
 ````python
  ! pip install opencv-python
  ! pip install pafy
  ! pip install soundfile
  #! pip install subprocess
  ! pip install youtube_dl
  ````
  <br/><br/>
  
 edit /usr/local/lib/python3.7/dist-packages/pafy/backend_youtube_dl.py 
 change  line 54  self._dislikes = self._ydl_info['dislike_count'] 
 to self._dislikes = 0
 <br/>
 restart and import again
 <br/><br/>
 
 count the number of videos of 8 classes same as VAS dataset, running following code
 ````python
  #labels_in_VAS = ['baby', 'cough', 'dog', 'drum', 'gun', 'fireworks', 'hammer', 'sneeze']
  with open('/content/drive/MyDrive/vggsound.csv') as f:
     lines = f.readlines()
  label_list = []
  for line in lines:
      line = line.strip().split(',')
     if line[2] == 'label in VGGSound' or ....:
        label_list.append(line[:2])
  print ('label_in_VAS:',len(label_list))
  ````
  <br/><br/>
  
  read video_id and start_time(s) from vggsound.csv
  ````python
  with open('/content/drive/MyDrive/vggsound.csv') as f:
    lines = f.readlines()
    fireworks_dict = {}
    for line in lines:
      line = line.strip().split(',')
      if line[2] == 'fireworks banging' :#or line[2] == 'lighting firecrackers' :
        fireworks_list.append(line[:2])
        fireworks_dict.update({line[0]:line[1]})
  print(len(fireworks_list), len(fireworks_dict))
  ````
  <br/><br/>
  
  download audio from YouTube
  ````python
  for i in range(len(label_list)):
  f1 = open('../error_log.txt','a+')
  f2 = open('../correct_log.txt','a+')
  #Get youtube-id
  ytid, ts_start = label_list[i]
  duration = 10
  ts_end = int(ts_start) + duration
  #Get the URL to the video page
  video_page_url = 'https://www.youtube.com/watch?v={}'.format(ytid)
  try:
    #Get the direct URLs to the videos with best audio and with best video (with audio)
    video = pafy.new(video_page_url)
    print('success')
    best_audio = video.getbestaudio()
    best_audio_url = best_audio.url
    audio_filepath = os.path.join('../', ytid + '.' + audio_codec)
    #Download the audio
    audio_dl_args = [ffmpeg_path, '-n',
        '-ss', str(ts_start),    #The beginning of the trim window
        '-i', best_audio_url,    #Specify the input video URL
        '-t', str(duration),     #Specify the duration of the output
        '-vn',                   #Suppress the video stream
        '-ac', '2',              #Set the number of channels
        '-sample_fmt', 's16',    #Specify the bit depth
        #'-acodec', audio_codec,  #Specify the output encoding
        '-ar', '44100',          #Specify the audio sample rate
        audio_filepath]
    proc = sp.Popen(audio_dl_args, stdout=sp.PIPE, stderr=sp.PIPE)
    stdout, stderr = proc.communicate()
    if proc.returncode != 0:
        print(stderr)
        f1.write(ytid+'\n')
    else:
        print(i,'Done!',"Downloaded audio to " + audio_filepath)
        f2.write(ytid+'\n')
  except:
    f1.write(ytid+'\n')
  ````
  <br/><br/>
  
  
  transform audio as melspecvqgan
  ````python
  cnt = 0
  for i in range(len(labels)):
      label = labels[i]
      datalist_path = os.path.join(wav_dir, label, 'correct_log.txt')
      ids = 0
      for line in open(datalist_path):
          line = line.split('\n')[0]
          try:
              mel_path = os.path.join(mel_dir, label, line + '.npy')
              data = np.load(mel_path)
              #plotsub figure
              if ids%150==0:
                  plt.savefig('../%s/test_sub_%d.pdf'
                              %(label,cnt), dpi=300)
                  plt.close()
                  plt.figure(figsize=(3,100))
                  cnt += 1 
              plt.subplot(150,1,ids+1-(cnt-1)*150)
              plt.title(line, fontsize=5)
              plt.axis('off')
              plt.imshow(data)
          except:
              print('ERROR!')
          ids += 1
  ````
  <br/><br/>
  
  select clean audio by human, and save the the clean audio id in one txt
  <br/><br/>
  
  
  download video from Youtube, following the id in clean audio txt
  code like audio downloading code
  ````python
   best_video = video.getbestvideo()
   best_video_url = best_video.url
   
   video_dl_args = [ffmpeg_path, '-n',
        '-ss', str(ts_start),   # The beginning of the trim window
        '-i', best_video_url,   # Specify the input video URL
        '-t', str(duration),    # Specify the duration of the output
        '-f', video_container,  # Specify the format (container) of the video
        '-framerate', '22',     # Specify the framerate
        '-vcodec', 'h264',      # Specify the output encoding
        video_filepath]
   ````
  
  


  
  
  
  
  
