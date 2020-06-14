# 【cocoapods进阶】-shell一键发布私有库

## 背景
>最近一直做组件化相关的改造，相关的私有库一直在频繁修改代码，但是每一次修改完业务代码，按照私有库发布的步骤，提交完修改的步骤后都需要，修改podspec文件的版本号和发布功能，然后校验是否合法，最后再执行发布指令。

## 本篇文档使用范围，可以为你做什么
> 为了不再私有库发布繁琐，一个指令帮助你完成自动提交代码、提交commit内容、打tag、自自定义relese nodes、检验、发布任务

##可能需要了解我的demo信息
step 1： 我的私有库索引库 名字：MySpecialIndexRepository 
step 2:   我的代码私有库信息 名字：https://github.com/GEGAOZHAO-developer/mySpecialCodeRepository.git
step 3:   业务代码主要在代码私有库目录下core文件夹下面

## 可能需要的开发语言基础
shell || linux || oc

## 由于我的脚本注释很详细 我这边不再赘述，一直贴上脚本，给伙伴们参考
```
#!/bin/bash

###############git 提交所有的变动 start################
git add *

echo "\n请输入当前要提交的内容"
read commitText

git commit -m '${commitText}'


###############spec信息 start################
#当前目录
RootPath=`pwd`

#PodSpec文件的名字
PodSpecName="MySpecialIndexRepository.podspec"

#PodSpec文件路径
PodSpecPath=$RootPath/$PodSpecName

#repo的私有库的名字
Repo_Name="MySpecialIndexRepository"


###############git 校验tag################
echo "\n请输入sdk tag版本号（3个位置版本号，例如：0.0.1 / 1.0.0 / 99.99.99 / .....）"
read Tag_Version

#判断输入的Tag_Version 不为空
if [ -z $Tag_Version ];then
    #输入的是空
    echo "\n输入格式错误，请重新运行脚本，输入正确格式的sdk tag版本号\n"
else
    #不是空
    # release的正则
    reg='^[0-9]{1,4}\.[0-9]{1,4}\.[0-9]{1,4}$'

    echo "Tag_Version》》》》》》 "$Tag_Version

    if [[ "$Tag_Version" =~ $reg ]];then
        echo "恭喜你，输入的版本号，格式验证通过"
    else
        echo "\n输入格式错误，请重新运行脚本，输入正确格式的sdk tag版本号\n"
        exit 1
    fi
fi


################定义函数-start################
function changeSpecVersion() {
echo ${PodSpecPath}

    while read line
    do
        zhengze="^spec.version"
        if [[ "$line" =~ $zhengze ]];then
            echo "File:${line}"
            sed -i "" "s/${line}/spec.version      = \"$Tag_Version\"/g" $PodSpecPath
        fi
    done < $PodSpecPath
#    cat  $PodSpecPath
}

function changeDistributeContent() {
   echo "\n请输入$Repo_Name \t 的 release nodes"
   read releaseNodes
   
    while read line
    do
        zhengze="^spec.summary"
        if [[ "$line" =~ $zhengze ]];then
            echo "File:${line}"
            sed -i "" "s/${line}/spec.summary      = \"$releaseNodes\"/g" $PodSpecPath
        fi
    done < $PodSpecPath
#    cat  $PodSpecPath
}

function pushPodRepo() {  
    #pod 提交
    # 修改spec文件并push上去
    pod repo push $Repo_Name $PodSpecName --verbose --allow-warnings
    echo "\n\n 发布 成功 新的版本号为 $Tag_Version"
}

################定义函数-end################

TempTagListFile=$RootPath/taglist.txt
echo "TempTagListFile:" $TempTagListFile
git fetch --tags
git tag -l |sort -r > $TempTagListFile

Have_Tag="0"
while read line
do
    tagNumber=$line
    echo "tagNumber:" $tagNumber

    if [ $tagNumber == $Tag_Version ];then
    Have_Tag="1"
        break
    fi
done < $TempTagListFile

if [[ $Have_Tag == "1" ]];then
    echo "\ntag号 $Tag_Version 已经存在,请重新输入!"
else
    echo "\ntag号 $Tag_Version 符合一切要求......继续后续工作"
    #调用changeSpecVersion方法
    changeSpecVersion
fi

#spec修改发布内容
changeDistributeContent

#git 暂存
git add .

#git 下拉代码
git pull

#git 提交tag
git add .
git commit -m "archive $PodSpecName in $Date"
git push
git tag $Tag_Version
git push --tags

#git 发布
pushPodRepo
```

#### 可能脚本写的不太好，如果有大神，请指导下
简书地址：https://www.jianshu.com/p/b504fe40fed1

