#### cmake
[cmake](http://www.cnblogs.com/binbinjx/p/5626916.html)
#### 有道云笔记 
[Marddown 使用](http://www.jianshu.com/p/1561ce3817fc)

#### vim插件管理
[vundle](http://blog.chinaunix.net/uid-12845622-id-3151378.html)

[插件集合](http://blog.csdn.net/namecyf/article/details/7787479)


[ctags使用](http://blog.csdn.net/gangyanliang/article/details/6889860)
http://blog.chinaunix.net/uid-20874550-id-2412585.html


[cppcomplete](http://blog.csdn.net/duguteng/article/details/7417181)

[文件头生成](http://www.cnblogs.com/OneFri/p/6077326.html)



```
"SET Comment START
autocmd BufNewFile *.h,*.c,*.cpp,*.cc exec ":call SetComment()" |normal 10Go

func SetComment()
"  if expand("%:e") == 'c'
"      call setline(1, '//c file')
"  elseif expand("%:e") == 'h'
"      call setline(1, '//head file')
"  elseif expand("%:e") == 'cpp'
"      call setline(1, '//C++ file')
"  endif

  call append(1, '//***********************************************')
  call append(2, '//')
  call append(3, '//      Filename: '.expand("%"))
  call append(4, '//')
  call append(5, '//      Author: wanmingxiang@baidu.com')
  call append(6, '//      Description: ---')
  call append(7, '//      Create: '.strftime("%Y-%m-%d %H:%M:%S"))
  call append(8, '//      Last Modified: '.strftime("%Y-%m-%d %H:%M:%S"))
  call append(9, '//***********************************************')
endfunc
map <F2> :call SetComment()<CR>:10<CR>o
"SET Comment END

"SET Last Modified Time START
func DataInsert()
  call cursor(9,1)
  if search ('Last Modified') != 0
      let line = line('.')
      call setline(line, '//      Last Modified: '.strftime("%Y-%m-%d %H:%M:%S"))
  endif
endfunc

autocmd FileWritePre,BufWritePre *.c,*.h,*.cpp ks|call DataInsert() |'s
"SET Last Modified Time END
```

