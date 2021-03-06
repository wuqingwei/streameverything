#summary Instructions for creating a streaming server at home to stream all your videos anywhere.

= Introduction =

This document will detail how to setup your own streaming server using Ubuntu 9.10, Apache2, Darwin Streaming Server, and VLC.


== Install Ubuntu 9.10 ==

This document assumes you are using Ubuntu 9.10 (32-bit) Server.  Other versions of Linux will most likely work, but I haven't tested them.

  # Download [http://www.ubuntu.com/getubuntu/download Ubuntu 9.10].
  # Install Ubuntu to a dedicated server, or to a virtual machine using your favorite virtualization platform.  Personally, I use [http://www.virtualbox.org VirtualBox].  I didn't select any packages during the install, as I like to install only what I need from the command line.

== Install Needed Software ==

{{{
sudo wget http://www.medibuntu.org/sources.list.d/`lsb_release -cs`.list --output-document=/etc/apt/sources.list.d/medibuntu.list
sudo apt-get -q update
sudo apt-get --yes -q --allow-unauthenticated install medibuntu-keyring
sudo apt-get -q update
sudo apt-get upgrade
sudo aptitude install apache2 openssh-server samba smbfs php5-ffmpeg libapache2-mod-php5 ffmpeg libavcodec-extra-52 vlc build-essential
}}}

== Install Darwin Streaming Server ==

{{{
sudo addgroup --system qtss
sudo adduser --system --no-create-home --ingroup qtss qtss
wget http://streameverything.googlecode.com/files/DarwinStreamingSrvr6.0.3-Source.tar
tar -xvf DarwinStreamingSrvr6.0.3-Source.tar
wget http://streameverything.googlecode.com/files/dss-6.0.3.patch
patch -p0 < dss-6.0.3.patch
http://streameverything.googlecode.com/files/dss-hh-20080728-1.patch
patch -p0 < dss-hh-20081021-1.patch
cd DarwinStreamingSrvr6.0.3-Source
mv Install Install.orig
wget http://streameverything.googlecode.com/files/Install
chmod +x Install
./Buildit
sudo ./Install
wget http://streameverything.googlecode.com/files/darwin-streaming-server
chmod +x darwin-streaming-server
sudo cp darwin-streaming-server /etc/init.d/darwin-streaming-server
sudo update-rc.d darwin-streaming-server defaults
}}}

  * At this point, you should browse to http://servername:1220/ and finish setting up Darwin Streaming Server. Just follow the prompts, it is pretty straight forward.
  * Next, test that you can stream an existing file by pointing your media player at rtsp://servername/sample_100kbit.mp4 or rtsp://servername/sample_50kbit.3gp depending on your player.  If neither of these work, stop here and figure out what went wrong.

== Testing VLC ==

  # Make sure the current user can write to /usr/local/movies.
  # Find the video you want to stream.
{{{
vlc /path/to/video --sout='#transcode{soverlay,ab=48,samplerate=44100,channels=1,acodec=mp4a,vcodec=h264,width=512,height=288,vfilter=\"canvas{width=512,height=288,aspect=16:9}\",fps=25,vb=384,venc=x264{vbv-bufsize=200,partitions=all,level=12,no-cabac,subme=7,threads=4,ref=2,mixed-refs=1,bframes=0,min-keyint=1,keyint=50,qpmax=51}}:gather:rtp{mp4a-latm,dst=127.0.0.1,port-audio=20000,port-video=20002,ttl=127,sdp=file:/usr/local/movies/movie.sdp}'
}}}
_These settings work for streaming to the Motorola DROID.  You may need to tweak them for your destination device._
  # Once you have started VLC, point your media player at rtsp://servername/movie.sdp.  You should see your video playing.

== Set Up Apache ==
  # Browse to http://servername/ to ensure your Apache setup is working. If you don't see a message saying "It Works!", stop and figure out what went wrong.
  # Download the [http://code.google.com/p/streameverything/source/browse/trunk/index.php index.php] file from this site, and place it in /var/www.
  # Restart Apache to make sure PHP is loaded: `sudo /etc/init.d/apache2 reload`.
  # Ensure your www-data user can write to /usr/local/movies.
  # Edit the index.php file: set the $basedir variable to be the path to your videos.  Set the $transcode variable to be whatever you needed to tweak the vlc settings to be.

== Try it all out ==

  # Browse to http://servername/index.php.
  # Browse until you find a video to play.
  # Click the video; this will start VLC in the background, transcoding the video.
  # Click the Watch Now link.  This will open the video in your player.  Be sure to click the stop button when you are done, otherwise VLC will continue to run, using up CPU cycles.

== Limitations ==

  * This only allows one video to be transcoded and streamed at a time.  This works great for personal use.
  * Cannot stream DVDs yet.
  * Cannot skip, fast-forward, rewind, etc.

== Troubleshooting ==

Your firewall needs the following ports open: TCP 554, UDP 6970:6999.

If you are behind a NAT firewall, you need to change the following line in streamingserver.xml:
{{{
<PREF NAME="alt_transport_src_ipaddr" ></PREF>
}}}
to (replace 123.45.67.8 with your public IP address):
{{{
<PREF NAME="alt_transport_src_ipaddr" >123.45.67.8</PREF>
}}}
The streaming server must be restarted in order for this to take effect.  If you do not have a static IP address then the streaming server needs to be reconfigured and restarted every time your IP address changes.
