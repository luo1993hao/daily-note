1.exec 





#!/usr/bin/env bash
#1. 由于项目module过多,构建后不利于发布,利用此脚本
#   通过脚本将编译好的jar,war,tar.gz文件查找出单独存放
### 2. clean 临时文件

PROJECT_PATH=$(dirname $0)


function build()
{
    mkdir -p ${PROJECT_PATH}/target
    mkdir -p ${PROJECT_PATH}/target/zip
    mkdir -p ${PROJECT_PATH}/target/jar
    mkdir -p ${PROJECT_PATH}/target/war

    #mvn clean package -Pdev
    find  ${PROJECT_PATH} -iname *lng*tar.gz* -exec cp {} ./target/zip \;
    find  ${PROJECT_PATH} -iname *lng*\.war* -exec cp {} ./target/war \;
    find  ${PROJECT_PATH} -iname *lng*jar* -exec cp {} ./target/jar \;

    #解压所有tar.gz
    cd ${PROJECT_PATH}/target/zip
    for tar in *.tar.gz;  do tar xvf $tar; done

    # 循环启动 rpc-service (目前start service.... 可能会阻塞)
    #find . -iname start.sh -exec sh {} \;

    #mvn -f lng-upms/lng-upms-server/pom.xml jetty:run-war -Pdev;
    #mvn -f lng-cms/lng-cms-admin/pom.xml  jetty:run-war -Pdev;
    #echo `find . -iname "deploy.sh" |xargs ls -l |awk '{print $9}'` >hehe.txt
    #find . -iname "deploy.sh" -exec ls -l {} \;

    build_start_shell
    build_war_start_shell
}

#target/zip/目录下 生成start 脚本
function build_start_shell(){

    cd ${PROJECT_PATH}/target

    for file in `find . -type f -iname "start.sh"`;
    do
#        echo "${file}"
        SHELL_FILE_NAME=`echo "${file}" | sed 's/.\/zip\/\(.*\)\/bin\/start\.sh$/\1/'`
#        echo "SHELL_FILE_NAME:${SHELL_FILE_NAME}"
        echo "create ${SHELL_FILE_NAME}.sh is ok !!!"
        echo "#!/bin/bash\n sh " ${file} > ./${SHELL_FILE_NAME}.sh
    done

}

function build_war_start_shell(){
    cd ${PROJECT_PATH}/

    for file in `find . -type f -iname "pom.xml" | xargs grep ">war<" | awk -F : '{print $1}'`;
    do
        SHELL_FILE_NAME=`echo "${file}" | sed "s/.\/\(.*\)\/\(.*\)\/pom\.xml/\2/"`
        echo "create war-${SHELL_FILE_NAME}.sh is ok !!!"
        echo "#!/bin/bash" > ./target/war-${SHELL_FILE_NAME}.sh
        echo "mvn -f .${file} jetty:run-war -Pdev" >> ./target/war-${SHELL_FILE_NAME}.sh
    done

}

# WARN mark: shell 函数命名不能包含"-"符号

function clean_dubbo_cache()
{
    find . -iname "*.cache" | xargs rm -rf
    find . -iname "*.lock" | xargs rm -rf
    find . -iname "*.log" | xargs rm -rf
    find . -type d -iname "target" | xargs rm -rf
#   清理log文件,排除.git下的log
    find . -type d -iname "logs" | grep -v "git" | xargs rm -rf

#    rm -rf ${PROJECT_PATH}/*.cache
#    rm -rf ${PROJECT_PATH}/*.lock
}

#根据首个参数做逻辑判断
case $1 in
         'build')
           build
         ;;
         'clean')
           clean_dubbo_cache
         ;;
         'build_start_shell')
           build_start_shell
         ;;
         'build_war_start_shell')
           build_war_start_shell
         ;;
         *)
           echo "not avalid arguments"
         ;;
esac
