---
layout: page
title:  "Crashing Browsers When Downloading Big Files"
date:   2020-12-28 17:30 +0200
categories: linux
---

I've experienced this funny problem that eventually turned out to be a misconfiguration issue done by none else but myself :) I want to share this bit in case I or someone else runs into it in the future.

I wanted to download an iso file with Fedora. Such files are usually at least a couple of hundreds megabytes, Fedora is about 1.9 GB. After a couple of hundreds megabytes (don't know exactly when, just that the download was never finished) Firefox crashed. Not great, not terrible. First, I repeated the same action, perhaps just something unexpected that would not repeat itself. Another crash. Alright, time to change a strategy. Let't try Chromium. Same issue. Brave. Same issue. Hm.

Looking into a log, it looked like so:

```
Dec 28 15:58:36 pavel-pc systemd-coredump[29648]: [🡕] Process 29167 (firefox) of user 1000 dumped core.
                                                  
                                                  Stack trace of thread 29644:
                                                  #0  0x00007fd98e128f6f __write (libpthread.so.0 + 0x12f6f)
                                                  #1  0x00007fd98d1ab3aa n/a (libnspr4.so + 0x253aa)
                                                  #2  0x00007fd98d1ac59e n/a (libnspr4.so + 0x2659e)
                                                  #3  0x00007fd98d1ad412 n/a (libnspr4.so + 0x27412)
                                                  #4  0x00007fd9871dd014 n/a (libxul.so + 0x4f51014)
                                                  #5  0x00007fd986b29b2b n/a (libxul.so + 0x489db2b)
                                                  #6  0x00007fd986b29ad2 n/a (libxul.so + 0x489dad2)
                                                  #7  0x00007fd98ac591cc n/a (libxul.so + 0x89cd1cc)
                                                  #8  0x00007fd9869a8ba5 n/a (libxul.so + 0x471cba5)
                                                  #9  0x00007fd98ac5914b n/a (libxul.so + 0x89cd14b)
                                                  #10 0x00007fd9869ed9e7 n/a (libxul.so + 0x47619e7)
                                                  #11 0x00007fd9869ed8e7 n/a (libxul.so + 0x47618e7)
                                                  #12 0x00007fd986aa3d98 n/a (libxul.so + 0x4817d98)
                                                  #13 0x00007fd98622c3ca n/a (libxul.so + 0x3fa03ca)
                                                  #14 0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #15 0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #16 0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #17 0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #18 0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #19 0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #20 0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29167:
                                                  #0  0x00007fd98dcff46f __poll (libc.so.6 + 0xf546f)
                                                  #1  0x00007fd988dfc6ba n/a (libxul.so + 0x6b706ba)
                                                  #2  0x00007fd98c70075f n/a (libglib-2.0.so.0 + 0xa675f)
                                                  #3  0x00007fd98c6ab121 g_main_context_iteration (libglib-2.0.so.0 + 0x51121)
                                                  #4  0x00007fd985aa7529 n/a (libxul.so + 0x381b529)
                                                  #5  0x00007fd985b57bc8 n/a (libxul.so + 0x38cbbc8)
                                                  #6  0x00007fd985b5592e n/a (libxul.so + 0x38c992e)
                                                  #7  0x00007fd985bc6b4b n/a (libxul.so + 0x393ab4b)
                                                  #8  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #9  0x00007fd988db12a6 n/a (libxul.so + 0x6b252a6)
                                                  #10 0x00007fd986f4dc60 n/a (libxul.so + 0x4cc1c60)
                                                  #11 0x00007fd98672d78b n/a (libxul.so + 0x44a178b)
                                                  #12 0x00007fd98670d89a n/a (libxul.so + 0x448189a)
                                                  #13 0x00007fd98670caa9 n/a (libxul.so + 0x4480aa9)
                                                  #14 0x000055a4c63f2e87 n/a (firefox + 0x1fe87)
                                                  #15 0x00007fd98dc32152 __libc_start_main (libc.so.6 + 0x28152)
                                                  #16 0x000055a4c64521ae _start (firefox + 0x7f1ae)
                                                  
                                                  Stack trace of thread 29171:
                                                  #0  0x00007fd98dcff46f __poll (libc.so.6 + 0xf546f)
                                                  #1  0x00007fd98c70075f n/a (libglib-2.0.so.0 + 0xa675f)
                                                  #2  0x00007fd98c6ab121 g_main_context_iteration (libglib-2.0.so.0 + 0x51121)
                                                  #3  0x00007fd98c6ab172 n/a (libglib-2.0.so.0 + 0x51172)
                                                  #4  0x00007fd98c6d9ce1 n/a (libglib-2.0.so.0 + 0x7fce1)
                                                  #5  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #6  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29300:
                                                  #0  0x00007fd98dd04d5d syscall (libc.so.6 + 0xfad5d)
                                                  #1  0x00007fd985fd9399 n/a (libxul.so + 0x3d4d399)
                                                  #2  0x00007fd985fd874d _ZN13tokio_reactor7Reactor4turn17h3bff0ea57286c5bcE (libxul.so + 0x3d4c74d)
                                                  #3  0x00007fd9864fc933 n/a (libxul.so + 0x4270933)
                                                  #4  0x00007fd987051d9a n/a (libxul.so + 0x4dc5d9a)
                                                  #5  0x00007fd987051c21 n/a (libxul.so + 0x4dc5c21)
                                                  #6  0x00007fd98abdefda n/a (libxul.so + 0x8952fda)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29172:
                                                  #0  0x00007fd98dcff46f __poll (libc.so.6 + 0xf546f)
                                                  #1  0x00007fd98c70075f n/a (libglib-2.0.so.0 + 0xa675f)
                                                  #2  0x00007fd98c6abe63 g_main_loop_run (libglib-2.0.so.0 + 0x51e63)
                                                  #3  0x00007fd98c547fe8 n/a (libgio-2.0.so.0 + 0x101fe8)
                                                  #4  0x00007fd98c6d9ce1 n/a (libglib-2.0.so.0 + 0x7fce1)
                                                  #5  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #6  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29178:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd986355e38 n/a (libxul.so + 0x40c9e38)
                                                  #3  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #4  0x00007fd9874ceb71 n/a (libxul.so + 0x5242b71)
                                                  #5  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #6  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #7  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #8  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #9  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29177:
                                                  #0  0x00007fd98dd04d5d syscall (libc.so.6 + 0xfad5d)
                                                  #1  0x00007fd985fd535e n/a (libxul.so + 0x3d4935e)
                                                  #2  0x00007fd985fd47b7 n/a (libxul.so + 0x3d487b7)
                                                  #3  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #4  0x00007fd986e4d77b n/a (libxul.so + 0x4bc177b)
                                                  #5  0x00007fd98aca59e7 n/a (libxul.so + 0x8a199e7)
                                                  #6  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #7  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29184:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x00007fd98d1a9fb7 n/a (libnspr4.so + 0x23fb7)
                                                  #2  0x00007fd98d1aa47d PR_WaitCondVar (libnspr4.so + 0x2447d)
                                                  #3  0x00007fd98771f78c n/a (libxul.so + 0x549378c)
                                                  #4  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #5  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #6  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29181:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29185:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643be31 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e31)
                                                  #2  0x00007fd986e18c0d n/a (libxul.so + 0x4b8cc0d)
                                                  #3  0x00007fd98af8ce43 n/a (libxul.so + 0x8d00e43)
                                                  #4  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #5  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29196:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29186:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643be31 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e31)
                                                  #2  0x00007fd986e18c0d n/a (libxul.so + 0x4b8cc0d)
                                                  #3  0x00007fd98af8ce43 n/a (libxul.so + 0x8d00e43)
                                                  #4  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #5  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29179:
                                                  #0  0x00007fd98dcff46f __poll (libc.so.6 + 0xf546f)
                                                  #1  0x00007fd986b48569 n/a (libxul.so + 0x48bc569)
                                                  #2  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29202:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29198:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29187:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643be31 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e31)
                                                  #2  0x00007fd986e18c0d n/a (libxul.so + 0x4b8cc0d)
                                                  #3  0x00007fd98af8ce43 n/a (libxul.so + 0x8d00e43)
                                                  #4  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #5  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29188:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643be31 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e31)
                                                  #2  0x00007fd986e18c0d n/a (libxul.so + 0x4b8cc0d)
                                                  #3  0x00007fd98af8ce43 n/a (libxul.so + 0x8d00e43)
                                                  #4  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #5  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29207:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd98665cf04 n/a (libxul.so + 0x43d0f04)
                                                  #3  0x00007fd98665be3b n/a (libxul.so + 0x43cfe3b)
                                                  #4  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #5  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #6  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #7  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #8  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #9  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #10 0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29209:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29214:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x00007fd9859fed2d _ZN10rayon_core5sleep5Sleep5sleep17ha0af856bab906664E (libxul.so + 0x3772d2d)
                                                  #2  0x00007fd9859fda98 n/a (libxul.so + 0x3771a98)
                                                  #3  0x00007fd986cbba0a _ZN10rayon_core8registry13ThreadBuilder3run17h0645d6b8271ede19E (libxul.so + 0x4a2fa0a)
                                                  #4  0x00007fd986cbb0c1 n/a (libxul.so + 0x4a2f0c1)
                                                  #5  0x00007fd986cbaff3 n/a (libxul.so + 0x4a2eff3)
                                                  #6  0x00007fd98abdefda n/a (libxul.so + 0x8952fda)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29200:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29215:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x00007fd9859fed2d _ZN10rayon_core5sleep5Sleep5sleep17ha0af856bab906664E (libxul.so + 0x3772d2d)
                                                  #2  0x00007fd9859fda98 n/a (libxul.so + 0x3771a98)
                                                  #3  0x00007fd986cbba0a _ZN10rayon_core8registry13ThreadBuilder3run17h0645d6b8271ede19E (libxul.so + 0x4a2fa0a)
                                                  #4  0x00007fd986cbb0c1 n/a (libxul.so + 0x4a2f0c1)
                                                  #5  0x00007fd986cbaff3 n/a (libxul.so + 0x4a2eff3)
                                                  #6  0x00007fd98abdefda n/a (libxul.so + 0x8952fda)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29201:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29277:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29213:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x00007fd9859fed2d _ZN10rayon_core5sleep5Sleep5sleep17ha0af856bab906664E (libxul.so + 0x3772d2d)
                                                  #2  0x00007fd9859fda98 n/a (libxul.so + 0x3771a98)
                                                  #3  0x00007fd986cbba0a _ZN10rayon_core8registry13ThreadBuilder3run17h0645d6b8271ede19E (libxul.so + 0x4a2fa0a)
                                                  #4  0x00007fd986cbb0c1 n/a (libxul.so + 0x4a2f0c1)
                                                  #5  0x00007fd986cbaff3 n/a (libxul.so + 0x4a2eff3)
                                                  #6  0x00007fd98abdefda n/a (libxul.so + 0x8952fda)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29301:
                                                  #0  0x00007fd98dd04d5d syscall (libc.so.6 + 0xfad5d)
                                                  #1  0x00007fd985fd9399 n/a (libxul.so + 0x3d4d399)
                                                  #2  0x00007fd985fd874d _ZN13tokio_reactor7Reactor4turn17h3bff0ea57286c5bcE (libxul.so + 0x3d4c74d)
                                                  #3  0x00007fd9864fc933 n/a (libxul.so + 0x4270933)
                                                  #4  0x00007fd9864f7d6a n/a (libxul.so + 0x426bd6a)
                                                  #5  0x00007fd9864f7bf1 n/a (libxul.so + 0x426bbf1)
                                                  #6  0x00007fd98abdefda n/a (libxul.so + 0x8952fda)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29278:
                                                  #0  0x00007fd98dcff46f __poll (libc.so.6 + 0xf546f)
                                                  #1  0x00007fd986f479b9 n/a (libxul.so + 0x4cbb9b9)
                                                  #2  0x00007fd986f47948 n/a (libxul.so + 0x4cbb948)
                                                  #3  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #4  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29280:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29313:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd98717c329 n/a (libxul.so + 0x4ef0329)
                                                  #3  0x00007fd98e002c24 execute_native_thread_routine (libstdc++.so.6 + 0xcfc24)
                                                  #4  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #5  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29287:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29314:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd98717c329 n/a (libxul.so + 0x4ef0329)
                                                  #3  0x00007fd98e002c24 execute_native_thread_routine (libstdc++.so.6 + 0xcfc24)
                                                  #4  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #5  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29279:
                                                  #0  0x00007fd98e129d3d recvmsg (libpthread.so.0 + 0x13d3d)
                                                  #1  0x00007fd98712a2d1 n/a (libxul.so + 0x4e9e2d1)
                                                  #2  0x00007fd98aca59e7 n/a (libxul.so + 0x8a199e7)
                                                  #3  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #4  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29312:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd98717c329 n/a (libxul.so + 0x4ef0329)
                                                  #3  0x00007fd98e002c24 execute_native_thread_routine (libstdc++.so.6 + 0xcfc24)
                                                  #4  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #5  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29319:
                                                  #0  0x00007fd98dcff46f __poll (libc.so.6 + 0xf546f)
                                                  #1  0x00007fd98c70075f n/a (libglib-2.0.so.0 + 0xa675f)
                                                  #2  0x00007fd98c6ab121 g_main_context_iteration (libglib-2.0.so.0 + 0x51121)
                                                  #3  0x00007fd96e01cc0e n/a (libdconfsettings.so + 0x5c0e)
                                                  #4  0x00007fd98c6d9ce1 n/a (libglib-2.0.so.0 + 0x7fce1)
                                                  #5  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #6  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29283:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29320:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29322:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd98622c2d7 n/a (libxul.so + 0x3fa02d7)
                                                  #3  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #4  0x00007fd9874ceb71 n/a (libxul.so + 0x5242b71)
                                                  #5  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #6  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #7  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #8  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #9  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29286:
                                                  #0  0x00007fd98e129d3d recvmsg (libpthread.so.0 + 0x13d3d)
                                                  #1  0x00007fd9870e3951 n/a (libxul.so + 0x4e57951)
                                                  #2  0x00007fd98aca59e7 n/a (libxul.so + 0x8a199e7)
                                                  #3  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #4  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29369:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd98665cf04 n/a (libxul.so + 0x43d0f04)
                                                  #3  0x00007fd98665be3b n/a (libxul.so + 0x43cfe3b)
                                                  #4  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #5  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #6  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #7  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #8  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #9  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #10 0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29325:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29321:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29326:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643be31 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e31)
                                                  #2  0x00007fd986f7b0ea n/a (libxul.so + 0x4cef0ea)
                                                  #3  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #4  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #5  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29422:
                                                  #0  0x00007fd98e129d3d recvmsg (libpthread.so.0 + 0x13d3d)
                                                  #1  0x00007fd9870e3951 n/a (libxul.so + 0x4e57951)
                                                  #2  0x00007fd98aca59e7 n/a (libxul.so + 0x8a199e7)
                                                  #3  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #4  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29330:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29329:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd98622c2d7 n/a (libxul.so + 0x3fa02d7)
                                                  #3  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #4  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #5  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #6  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #7  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #8  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #9  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29580:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd98622c2d7 n/a (libxul.so + 0x3fa02d7)
                                                  #3  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #4  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #5  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #6  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #7  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #8  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #9  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29331:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29417:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29597:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd98622c2d7 n/a (libxul.so + 0x3fa02d7)
                                                  #3  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #4  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #5  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #6  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #7  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #8  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #9  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29332:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29454:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd98622c2d7 n/a (libxul.so + 0x3fa02d7)
                                                  #3  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #4  0x00007fd9874ceb71 n/a (libxul.so + 0x5242b71)
                                                  #5  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #6  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #7  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #8  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #9  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29598:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd98622c2d7 n/a (libxul.so + 0x3fa02d7)
                                                  #3  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #4  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #5  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #6  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #7  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #8  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #9  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29552:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29336:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd986ec9fe5 n/a (libxul.so + 0x4c3dfe5)
                                                  #3  0x00007fd98ac767dc n/a (libxul.so + 0x89ea7dc)
                                                  #4  0x00007fd98622c3ca n/a (libxul.so + 0x3fa03ca)
                                                  #5  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #6  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #7  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #8  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #9  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #10 0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #11 0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29601:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd98622c2d7 n/a (libxul.so + 0x3fa02d7)
                                                  #3  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #4  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #5  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #6  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #7  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #8  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #9  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29342:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29180:
                                                  #0  0x00007fd98dcff46f __poll (libc.so.6 + 0xf546f)
                                                  #1  0x00007fd98d1abeff n/a (libnspr4.so + 0x25eff)
                                                  #2  0x00007fd986023636 n/a (libxul.so + 0x3d97636)
                                                  #3  0x00007fd98602300a n/a (libxul.so + 0x3d9700a)
                                                  #4  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #5  0x00007fd9874ceb71 n/a (libxul.so + 0x5242b71)
                                                  #6  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #7  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #8  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #9  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #10 0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29355:
                                                  #0  0x00007fd98dd04d5d syscall (libc.so.6 + 0xfad5d)
                                                  #1  0x00007fd986ffeeba _ZN3std6thread4park17h5f5a761a5966bc85E (libxul.so + 0x4d72eba)
                                                  #2  0x00007fd986ffed97 n/a (libxul.so + 0x4d72d97)
                                                  #3  0x00007fd986ffea92 n/a (libxul.so + 0x4d72a92)
                                                  #4  0x00007fd986ffda2f n/a (libxul.so + 0x4d71a2f)
                                                  #5  0x00007fd986ffd03f n/a (libxul.so + 0x4d7103f)
                                                  #6  0x00007fd98abdefda n/a (libxul.so + 0x8952fda)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29384:
                                                  #0  0x00007fd98dcd1c51 clock_nanosleep@@GLIBC_2.17 (libc.so.6 + 0xc7c51)
                                                  #1  0x00007fd98dcd7137 __nanosleep (libc.so.6 + 0xcd137)
                                                  #2  0x00007fd9862fa530 _ZN3std6thread5sleep17hf2e3c69a59175fecE (libxul.so + 0x406e530)
                                                  #3  0x00007fd9862fa47c n/a (libxul.so + 0x406e47c)
                                                  #4  0x00007fd9862fa435 n/a (libxul.so + 0x406e435)
                                                  #5  0x00007fd98abdefda n/a (libxul.so + 0x8952fda)
                                                  #6  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #7  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29390:
                                                  #0  0x00007fd98e129d3d recvmsg (libpthread.so.0 + 0x13d3d)
                                                  #1  0x00007fd9870e3951 n/a (libxul.so + 0x4e57951)
                                                  #2  0x00007fd98aca59e7 n/a (libxul.so + 0x8a199e7)
                                                  #3  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #4  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29415:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29480:
                                                  #0  0x00007fd98e129d3d recvmsg (libpthread.so.0 + 0x13d3d)
                                                  #1  0x00007fd9870e3951 n/a (libxul.so + 0x4e57951)
                                                  #2  0x00007fd98aca59e7 n/a (libxul.so + 0x8a199e7)
                                                  #3  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #4  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29596:
                                                  #0  0x00007fd98e1259c8 pthread_cond_timedwait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf9c8)
                                                  #1  0x000055a4c643be01 _ZN7mozilla6detail21ConditionVariableImpl8wait_forERNS0_9MutexImplERKNS_16BaseTimeDurationINS_27TimeDurationValueCalculatorEEE (firefox + 0x68e01)
                                                  #2  0x00007fd98622c2d7 n/a (libxul.so + 0x3fa02d7)
                                                  #3  0x00007fd985b56269 n/a (libxul.so + 0x38ca269)
                                                  #4  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #5  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #6  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #7  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #8  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #9  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
                                                  
                                                  Stack trace of thread 29637:
                                                  #0  0x00007fd98e1256a2 pthread_cond_wait@@GLIBC_2.3.2 (libpthread.so.0 + 0xf6a2)
                                                  #1  0x000055a4c643bcb7 _ZN7mozilla6detail21ConditionVariableImpl4waitERNS0_9MutexImplE (firefox + 0x68cb7)
                                                  #2  0x00007fd985b56af0 n/a (libxul.so + 0x38caaf0)
                                                  #3  0x00007fd9874cec28 n/a (libxul.so + 0x5242c28)
                                                  #4  0x00007fd9869de72d n/a (libxul.so + 0x475272d)
                                                  #5  0x00007fd9869de22c n/a (libxul.so + 0x475222c)
                                                  #6  0x00007fd98d1b1088 n/a (libnspr4.so + 0x2b088)
                                                  #7  0x00007fd98e11f3e9 start_thread (libpthread.so.0 + 0x93e9)
                                                  #8  0x00007fd98dd0a293 __clone (libc.so.6 + 0x100293)
Dec 28 15:58:37 pavel-pc systemd[1]: systemd-coredump@8-29647-0.service: Succeeded.
```

Similarly for Chromium and other browsers I tried (Brave for example).

There're a couple of things I tried. These might occasionally help but didn't in my case:

- turning off all browser add-ons and extensions (in Firefox 84: Help > Restart with Add-ons Disabled > Restart > Start in Safe Mode)
- reinstalling the browser
- deleting cache and everything possible from the broser

What was the issue on my side then?

I simply added my user into a new group some time ago:

```
# usermod -a -G users pavel
```

while at the same time having set up limits for `users` group:

```
$ cat /etc/security/limits.conf
...
@users       hard    fsize       1000000
...
```

The `fsize` rule was simply enforced when a file hit 1 GB mark. None of the browsers knew how to deal with it, so they all crashed in this situation.

I eventually realised I didn't need to have my user in the group, therefore removing the user from the group solved the problem:

```
# gpasswd -d pavel users
```
