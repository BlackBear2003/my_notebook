打算每个微服务都写一个dockerfile 然后再用dockercompose一起管理

docker build -t luke0125/accounter:0.1 -f ./dockerfile .

docker run -itd --name accounter -v /home/ubuntu/Accounter/file:/tmp/file -p 80:80 luke0125/accounter:0.1

docker push luke0125/accounter:TAGNAME

