# 导读
1. awk语法
2. awk正则
3. awk统计


# 1. awk语法



# 2. awk正则

# 3. 常用脚本
awk BEGIN{RS=EOF}'{gsub(/\n/,",");print}' fileName				//将多行合并成一行
awk -F ',' '{for(i=1; i<=NF; i++){print $i;}}' fileName 		//将一行分成多行
netstat -an|awk '/^tcp/{++s[$NF]}END{for(a in s)print a,s[a]}'	//查看服务器连接状态并汇总 

# 4. awk统计
