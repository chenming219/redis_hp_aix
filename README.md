# HP-UX AIX 编译环境

1. HP-UX:acc
2. AIX:xlc

# 新增环境变量

1. HP-UX:~/.profile 新增 export OS=HP-UX
2. AIX:~/.profile 新增 export OS=AIX

# 编译流程

1. $cd redis/deps/hiredis
2. $make
3. $cd redis/deps/lua/src
4. $make ${OS}
5. $cd redis/src
6. make
