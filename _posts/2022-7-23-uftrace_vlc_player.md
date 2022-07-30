---
title: uftrace를 사용한 분석 - vlc player
author: Wendy
categories: [Blogging, Uftrace]
tags: [study]
---


### VLC 빌드

#### 소스 가져오기

```console
git clone https://github.com/videolan/vlc.git
```

#### 필요 패키지 설치

WSL2 환경에서 다수의 패키지의 설치가 필요했다.
bootstrap과 configure 단계에서 에러 발생시 다음의 사이트에서 검색해서 찾을 수 있었다.

ubuntu 패키지 찾기 <https://packages.ubuntu.com/search?keywords=xcb&searchon=names&suite=jammy&section=all>


./bootstrap 단계에서 설치한 패키지

```console
sudo apt install flex bison autopoint gettext libtool autoconf
```
./configure 단계에서 설치한 패키지 (--disable-qt 옵션 사용)
```console
sudo apt install lua5.2 liblua5.2-dev libavcodec-dev libavutil-dev libavformat-dev libswscale-dev
sudo apt install liba52-dev libxcb-composite0-dev libalsa-ocaml-dev
sudo apt install libxcb-randr0-dev libxcb-shm0-dev libxcb-xkb-dev
```

#### vlc player 빌드

vlc player에 pg 옵션을 넣어 빌드!
``` console
./bootstrap
./configure --disable-qt CFLAGS="-pg"
./make 
```

### UFTRACE를 이용한 VLC 분석

#### graph 

##### task  
``` console
	$ uftrace graph --task
	========== TASK GRAPH ==========
	# TOTAL TIME   SELF TIME     TID     TASK NAME
		1.043  m   82.056 ms  [ 19833] : vlc
		1.043  m   22.826 us  [ 19836] :  +-vlc-exec-runner
		1.043  m    1.589 ms  [ 19837] :  +-vlc-exec-runner
		1.043  m   29.841 us  [ 19839] :  +-vlc-exec-runner
		1.043  m    3.003 ms  [ 19838] :  +-vlc-exec-runner
		1.043  m   18.807 us  [ 19840] :  +-vlc-exec-runner
		1.043  m  546.857 us  [ 19841] :  +-vlc-player-end
	   33.906 ms   22.701 ms  [ 19842] :  +-vlc-preparse
		1.043  m  348.612 us  [ 19843] :  +-vlc-cli-client
		1.021  m  387.175 ms  [ 19844] :  +-vlc-input
		1.021  m  383.284 ms  [ 19851] :  +-vlc-dec-video
		1.021  m  580.095 ms  [ 19852] :  +-vlc-dec-audio
		1.019  m    1.006  m  [ 19845] :  +-vlc-static
		1.043  m   61.519 ms  [ 19854] :  +-vlc-spu-prerend
		1.043  m    1.055 ms  [ 19855] :  +-vlc-window-x11
		1.043  m  408.717 us  [ 19856] :  +-vlc-timer
		1.020  m    3.686  s  [ 19873] :  +-vlc-vout
		1.019  m    1.006  m  [ 19846] :  +-vlc-static
		1.019  m    1.006  m  [ 19847] :  +-vlc-static
		1.019  m    1.006  m  [ 19848] :  +-vlc-static
		1.019  m    1.006  m  [ 19849] :  +-vlc-static
		1.019  m    1.006  m  [ 19850] :  +-vlc-static
```
 
 
#### replay 

- vlc-dec-video
위의 task 중 vlc-dec-video thread를 조금 더 알고 싶다.  
그런데 결과에 mutex가 너무 많아~서 mutex 는 제외하고 확인해본다.
``` console 
	$ uftrace replay --tid=19851 -N vlc_mutex* -T 'DecoderThread_ProcessInput@filter,backtrace,depth=5'
	# DURATION     TID     FUNCTION
	  backtrace [ 19851] | /* [ 0] DecoderThread */
				[ 19851] | DecoderThread_ProcessInput() {
	   0.055 us [ 19851] |   DecoderUpdatePreroll();
				[ 19851] |   DecoderThread_DecodeBlock() {
				[ 19851] |     vlc_object_get_tracer() {
				[ 19851] |       vlc_object_instance() {
	   0.139 us [ 19851] |         vlc_object_parent();
	   0.221 us [ 19851] |         vlc_object_parent();
	   0.152 us [ 19851] |         vlc_object_parent();
	   0.083 us [ 19851] |         vlc_object_parent();
	   0.979 us [ 19851] |       } /* vlc_object_instance */
	   0.034 us [ 19851] |       libvlc_priv();
	   1.177 us [ 19851] |     } /* vlc_object_get_tracer */
				[ 19851] |     DecodeVideo() {
				[ 19851] |       DecodeBlock() {
	   0.056 us [ 19851] |         filter_earlydropped_blocks();
	   0.534 us [ 19851] |         vlc_frame_Realloc();
	   1.238 us [ 19851] |         vlc_frame_Release();
	  96.708 us [ 19851] |       } /* DecodeBlock */
	  96.932 us [ 19851] |     } /* DecodeVideo */
	  98.796 us [ 19851] |   } /* DecoderThread_DecodeBlock */
	  99.964 us [ 19851] | } /* DecoderThread_ProcessInput */
	  backtrace [ 19851] | /* [ 0] DecoderThread */
				[ 19851] | DecoderThread_ProcessInput() {
	   0.104 us [ 19851] |   DecoderUpdatePreroll();
```
 
- vlc-input
``` console
	$ uftrace replay --tid=19844 -N vlc_mutex*
	# DURATION     TID     FUNCTION
				[ 19844] | Run() {
				[ 19844] |   vlc_thread_set_name() {
				[ 19844] |     /* linux:task-name (comm="vlc-input") */
	   1.757 us [ 19844] |   } /* vlc_thread_set_name */
	   0.103 us [ 19844] |   vlc_interrupt_set();
				[ 19844] |   Init() {
	   0.056 us [ 19844] |     input_priv();
				[ 19844] |     input_ChangeState() {
	   0.046 us [ 19844] |       input_priv();
	   0.040 us [ 19844] |       input_priv();
				[ 19844] |       input_SendEventState() {
				[ 19844] |         input_SendEvent() {
	   0.056 us [ 19844] |           input_priv();
				[ 19844] |           input_thread_Events() {
				[ 19844] |             vlc_player_input_HandleStateEvent() {
				[ 19844] |               vlc_player_input_HandleState() {
	   0.068 us [ 19844] |                 vlc_list_it_start();
	   0.041 us [ 19844] |                 vlc_list_it_continue();
	   0.061 us [ 19844] |                 vlc_list_it_next();
	   0.041 us [ 19844] |                 vlc_list_it_continue();
	   0.055 us [ 19844] |                 vlc_list_it_next();
	   0.049 us [ 19844] |                 vlc_list_it_continue();
				[ 19844] |                 player_on_state_changed() {
				[ 19844] |                   msg_print() {
				[ 19844] |                     msg_vprint() {
	   0.040 us [ 19844] |                       vlc_list_it_start();
	   0.046 us [ 19844] |                       vlc_list_it_continue();
				[ 19844] |                       cli_vprintf() {
				[ 19844] |                         cli_writev() {
	  16.744 us [ 19844] |                           vlc_writev();
	  20.204 us [ 19844] |                         } /* cli_writev */
	  21.418 us [ 19844] |                       } /* cli_vprintf */
	   0.064 us [ 19844] |                       vlc_list_it_next();
	   0.037 us [ 19844] |                       vlc_list_it_continue();
	  27.447 us [ 19844] |                     } /* msg_vprint */
	  27.586 us [ 19844] |                   } /* msg_print */
	  27.741 us [ 19844] |                 } /* player_on_state_changed */
```

#### report

- 가장 많이 불린 함수는?
``` console
		$ uftrace report -f all -s call
		Total time   Total avg   Total min   Total max   Self time    Self avg    Self min    Self max       Calls  Function
		==========  ==========  ==========  ==========  ==========  ==========  ==========  ==========  ==========  ====================
		74.468 ms    0.123 us    0.025 us   62.331 us   74.468 ms    0.123 us    0.025 us   62.331 us      604988  vlc_thread_id
		189.923 ms    0.458 us    0.103 us  226.244 us  131.843 ms    0.318 us    0.074 us  130.732 us      414036  vlc_mutex_held
		196.476 ms    1.031 us    0.220 us  227.229 us   75.129 ms    0.394 us    0.094 us  128.730 us      190424  vlc_mutex_trylock
		131.035 ms    0.688 us    0.168 us  177.329 us   40.653 ms    0.213 us    0.062 us  176.732 us      190424  vlc_mutex_unlock
		681.213 ms    3.613 us    0.308 us  212.013 ms   37.608 ms    0.199 us    0.059 us  129.470 us      188501  vlc_mutex_lock
		14.842 ms    0.141 us    0.026 us   73.956 us   14.842 ms    0.141 us    0.026 us   73.956 us      105224  fourcc_cmp
		11.186 ms    0.155 us    0.024 us  104.158 us   11.186 ms    0.155 us    0.024 us  104.158 us       71910  vlc_object_parent
		4.119 ms    0.061 us    0.024 us   34.994 us    4.119 ms    0.061 us    0.024 us   34.994 us       67420  MP4_rescale
		17.242 ms    0.277 us    0.096 us  103.390 us   13.443 ms    0.216 us    0.070 us  103.279 us       62034  MP4_rescale_mtime
		3.258 ms    0.067 us    0.024 us   29.562 us    3.258 ms    0.067 us    0.024 us   29.562 us       48173  vlc_tick_from_seci
		18.935 ms    0.393 us    0.109 us  134.327 us   15.676 ms    0.325 us    0.082 us  134.204 us       48173  vlc_tick_now
		3.466 ms    0.093 us    0.028 us   50.077 us    3.466 ms    0.093 us    0.028 us   50.077 us       36932  GCD
		2.334 ms    0.068 us    0.024 us   21.851 us    2.334 ms    0.068 us    0.024 us   21.851 us       34194  input_priv
		1.001  h   45.701 ms    1.354 us    1.043  m    1.001  h   45.701 ms    1.354 us    1.043  m       33619  linux:schedule
		23.596 ms    0.839 us    0.299 us  104.639 us    8.053 ms    0.286 us    0.098 us   73.451 us       28124  MP4_TrackGetDTSPTS
		9.644 ms    0.353 us    0.113 us  102.483 us    7.193 ms    0.263 us    0.081 us  102.312 us       27258  LCM
		1.723 ms    0.063 us    0.024 us   12.770 us    1.723 ms    0.063 us    0.024 us   12.770 us       27051  vlc_fifo_queue
```
- 비교를 해보자~ 
``` console	
	$ uftrace report --diff uftrace.data.old -f total,total-min,self-avg --diff-policy full
	#
	# uftrace diff
	#  [0] base: uftrace.data       (from uftrace record --srcline ./vlc ./sunrise.mp4)
	#  [1] diff: uftrace.data.old   (from uftrace record ./vlc ./sunrise.mp4)
	#
						 Total time (diff)                      Total min (diff)                       Self avg (diff)   Function
	   ===================================   ===================================   ===================================   ====================
		15.030  m    2.022  h    +2.006  h    42.994  s    4.008  m    +3.025  m    58.142  s    4.023  m    +3.024  m   exit
		 1.001  h    2.021  h    +1.020  h     1.354 us           -    -1.354 us    45.701 ms   94.813 ms   +49.112 ms   linux:schedule
		18.051  m    1.022  h    +1.003  h     0.388 us    0.382 us    -0.006 us     2.025 us    1.726 us    -0.299 us   vlc_futex_wait
		18.052  m    1.022  h    +1.003  h     0.108 us    0.106 us    -0.002 us    14.888 us    9.685 us    -5.203 us   sys_futex
		15.005  m    1.018  h    +1.003  h    56.737 us   18.658 us   -38.079 us     1.136 us    0.748 us    -0.388 us   vlc_cond_wait
		15.006  m    1.018  h    +1.003  h     0.467 us    0.470 us    +0.003 us     0.264 us    0.170 us    -0.094 us   vlc_atomic_wait
		 8.037  m    1.001  h   +17.004  m     1.043  m    5.008  m    +3.024  m     1.999 us    2.635 us    +0.636 us   ThreadRun
		 8.037  m    1.001  h   +17.004  m    42.627 ms   43.002 ms  +374.319 us     0.676 us    0.706 us    +0.030 us   QueueTake
		 1.043  m    5.008  m    +3.024  m     1.043  m    5.008  m    +3.024  m    10.128 ms   12.895 ms    +2.767 ms   cli_client_thread
		 1.043  m    5.008  m    +3.024  m     1.043  m    5.008  m    +3.024  m    10.606 us    6.500 us    -4.106 us   vlc_player_destructor_Thread
		 1.043  m    5.008  m    +3.024  m     1.043  m    5.008  m    +3.024  m    25.006 us   26.744 us    +1.738 us   sigwait
		 1.043  m    5.008  m    +3.024  m     1.043  m    5.008  m    +3.024  m    15.012 us    9.342 us    -5.670 us   main
		 1.043  m    5.008  m    +3.024  m     1.043  m    5.008  m    +3.024  m     2.025 ms    1.437 ms  -588.921 us   spu_PrerenderThread
		 1.043  m    5.008  m    +3.024  m     1.043  m    5.008  m    +3.024  m    26.145 us   10.792 us   -15.353 us   vlc_timer_thread
		 3.004  m    6.028  m    +3.024  m     1.020  m    1.020  m  -816.621 ms     5.141 ms    5.418 ms  +277.303 us   Thread
		 1.019  m   28.347  s   -51.258  s    13.959 us   31.061 us   +17.102 us     0.814 us    0.578 us    -0.236 us   DecoderThread_ProcessInput
		 1.019  m   28.337  s   -51.254  s    13.561 us   30.176 us   +16.615 us     0.610 us    0.427 us    -0.183 us   DecoderThread_DecodeBlock
		 1.019  m   28.328  s   -51.250  s    10.271 us   29.443 us   +19.172 us    59.192 us   41.302 us   -17.890 us   DecodeBlock
		 1.018  m   27.729  s   -51.146  s    66.771 us   55.661 us   -11.110 us     0.222 us    0.183 us    -0.039 us   DecodeVideo
		 1.020  m    2.011  m   +50.527  s    57.159 us   19.057 us   -38.102 us     0.337 us    0.263 us    -0.074 us   vlc_fifo_Wait
		 1.020  m    2.011  m   +50.527  s    56.910 us   18.792 us   -38.118 us     0.458 us    0.268 us    -0.190 us   vlc_queue_Wait
		 1.018  m   40.132  s   -37.999  s    10.171 us    9.671 us    -0.500 us    12.314 us    6.998 us    -5.316 us   lavc_dr_GetFrame
		 1.018  m   40.116  s   -37.988  s     8.660 us    7.828 us    -0.832 us     0.800 us    0.438 us    -0.362 us   decoder_NewPicture
		 1.018  m   40.115  s   -37.987  s     8.528 us    7.708 us    -0.820 us     1.539 us    1.026 us    -0.513 us   ModuleThread_NewVideoBuffer
		 1.018  m   40.112  s   -37.986  s     7.870 us    7.104 us    -0.766 us     1.653 us    1.104 us    -0.549 us   picture_pool_Wait
		 1.018  m   40.357  s   -37.952  s    28.149 us   24.503 us    -3.646 us     2.347 us    1.542 us    -0.805 us   lavc_GetFrame
		12.402  s    7.447  s    -4.955  s     3.275 ms    3.119 ms  -156.284 us   204.669 us  136.946 us   -67.723 us   PictureRender
		 1.004  m    1.008  m    +4.080  s     7.003 us    8.921 us    +1.918 us     0.720 us    0.530 us    -0.190 us   vlc_clock_Wait
		 3.045  m    3.048  m    +2.966  s     1.258 us    0.852 us    -0.406 us     1.253 us    0.959 us    -0.294 us   vlc_atomic_timedwait
		 3.045  m    3.048  m    +2.948  s     6.549 us    8.342 us    +1.793 us     1.298 us    1.035 us    -0.263 us   vlc_cond_timedwait
```
#### info

Todo) 여기에서 FLAGS가 의미는?
``` console
	$ uftrace info --task
	#         TIMESTAMP       FLAGS     TID    TASK              DATA SIZE
			165175.517868585  FS     [ 19833]  vlc                   3.797 MB
			165175.565699180         [ 19836]  vlc-exec-runner       0.001 MB
			165175.566791025         [ 19837]  vlc-run-prepars       0.117 MB
			165175.566866697         [ 19838]  vlc-exec-runner       0.038 MB
			165175.566988895         [ 19839]  vlc-exec-runner       0.001 MB
			165175.567439094         [ 19840]  vlc-exec-runner       0.001 MB
			165175.609405416         [ 19841]  vlc-player-end        0.015 MB
			165175.611223100         [ 19842]  vlc-preparse          1.320 MB
			165175.614638686         [ 19843]  vlc-cli-client        0.001 MB
			165175.615249899         [ 19844]  vlc-input            35.978 MB
			165175.661827535         [ 19845]  vlc-static            2.447 MB
			165175.661912350         [ 19846]  vlc-static            1.969 MB
			165175.661977935         [ 19847]  vlc-static            1.970 MB
			165175.662046004         [ 19848]  vlc-static            1.969 MB
			165175.662107140         [ 19849]  vlc-static            1.966 MB
			165175.662166497         [ 19850]  vlc-static            1.963 MB
			165175.662216979         [ 19851]  vlc-dec-video        15.106 MB
			165175.681335551         [ 19852]  vlc-dec-audio        25.087 MB
			165175.767731697         [ 19854]  vlc-spu-prerend       2.486 MB
			165175.773089153         [ 19855]  vlc-window-x11        0.023 MB
			165175.777119407         [ 19856]  vlc-timer             0.006 MB
			165175.853211605         [ 19873]  vlc-vout             28.484 MB
			165205.778313709  FS     [ 19874]  xdg-screensaver       0.553 MB
```

#### live

#### dump

#### recv

#### script
``` console
	$ uftrace script -S ../../uftrace/scripts/simple.py
	program begins...
	entry : main()
	entry : signal()
	exit  : signal()
	entry : signal()
	exit  : signal()
```

#### tui

- call graph에서 출력되는 session의 순서는 가장 동작이 많은 순서인가?
``` console
	 G  call Graph for session #1: vlc-static                                                                      
		call Graph for session #2: dash
		call Graph for session #3: mv
		call Graph for session #4: grep
		call Graph for session #5: sed
		call Graph for session #6: dash
	  TOTAL TIME : FUNCTION
		1.001  h : (1) vlc-static
		1.043  m :  ├▶(1) main
				 :  │
		8.037  m :  ├▶(5) ThreadRun
				 :  │
		1.043  m :  ├▶(1) vlc_player_destructor_Thread
				 :  │
	   33.906 ms :  ├▶(1) Preparse
				 :  │
		1.043  m :  ├▶(1) cli_client_thread
				 :  │
		1.021  m :  ├▶(1) Run
				 :  │
		2.042  m :  ├▶(2) DecoderThread                                                                            
				 :  │
	   94.506 ms :  ├▶(1) ffmpeg_GetFormat
				 :  │
		1.043  m :  ├▶(1) spu_PrerenderThread
				 :  │
		3.004  m :  ├▶(2) Thread
				 :  │
		1.043  m :  ├▶(1) vlc_timer_thread
				 :  │
		1.018  m :  └▶(1923) lavc_GetFrame
 ```

- DecoderThread 열어보자~
``` console
	  TOTAL TIME : FUNCTION
		2.042  m :  ├─(2) DecoderThread
		3.868 us :  │  ├─(2) vlc_thread_set_name
				 :  │  │
	   11.097 ms :  │  ├▶(5374) vlc_fifo_Lock
				 :  │  │
	   20.813 ms :  │  ├▶(11400) vlc_cond_signal
				 :  │  │
	   14.313 ms :  │  ├▶(5700) vlc_fifo_DequeueUnlocked
				 :  │  │
		1.020  m :  │  ├▶(328) vlc_fifo_Wait
				 :  │  │
		8.924 ms :  │  ├▶(5374) vlc_fifo_Unlock
				 :  │  │
		1.019  m :  │  ├─(5372) DecoderThread_ProcessInput
		5.929 ms :  │  │  ├▶(5370) vlc_mutex_lock
				 :  │  │  │
	  611.426 us :  │  │  ├─(5370) DecoderUpdatePreroll
				 :  │  │  │
		3.098 ms :  │  │  ├▶(5370) vlc_mutex_unlock
				 :  │  │  │
		1.019  m :  │  │  └─(5372) DecoderThread_DecodeBlock
		8.398 ms :  │  │     ├▶(5372) vlc_object_get_tracer
				 :  │  │     │
	  704.837 ms :  │  │     ├─(3448) DecodeAudio
	  704.002 ms :  │  │     │▶(3448) DecodeBlock
				 :  │  │     │
		1.018  m :  │  │     └─(1924) DecodeVideo
		1.018  m :  │  │      ▶(1924) DecodeBlock
				 :  │  │
		7.150 ms :  │  ├▶(5372) vlc_mutex_lock
				 :  │  │
		6.795 ms :  │  ├▶(5372) vlc_mutex_unlock
				 :  │  │
	  914.324 us :  │  ├▶(320) vlc_aout_stream_UpdateLatency
				 :  │  │
		1.959  s :  │  └▶(1) vlc_aout_stream_Drain
				 :  │
	   94.506 ms :  ├▶(1) ffmpeg_GetFormat
```


