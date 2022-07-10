---
layout: post
title:  "WSL2 settings"
date:   2022-07-10 09:00:00 +0900
categories: Environment WSL
---
*내가 자주 사용하는 WSL2 설정*
  
### WSL 외부 network 설정
Reference > <https://codeac.tistory.com/118>
  
WSL1과 달리 WSL2부터는 기본적으로 WSL과 외부 network가 연결되어 있지 않음.  
아래 script에서 $ports 부분에 forwarding 하고 싶은 port를 입력하고 .ps1으로 저장하고 실행시키면 됨.
- 관리자 권한 필요
- 부팅 시마다 수행해줘야 하기 때문에 service에서 등록하고 사용하면 편함.
  
```shell
$remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"  
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';  
  
if( $found ){
  $remoteport = $matches[0];
} else{
  echo "The Script Exited, the ip address of WSL 2 cannot be found";
  exit;
}
  
#[Ports]
  
#All the ports you want to forward separated by coma
$ports=@(80,443,10000,3000,5000);
  
  
#[Static ip]
#You can change the addr to your ip config to listen to a specific address
$addr='0.0.0.0';
$ports_a = $ports -join ",";
  
  
#Remove Firewall Exception Rules
iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";
  
#adding Exception Rules for inbound and outbound Rules
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";
  
for( $i = 0; $i -lt $ports.length; $i++ ){
  $port = $ports[$i];
  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport";
}
```
  
#### [Troubleshooting]
Powershell은 기본적으로 script 수행이 막혀있음.  
때문에 아래와 같은 오류가 발생하는 경우 policy를 변경해줘야 한다.
```console
+ CategoryInfo          : 보안 오류: (:) [], PSSecurityException
+ FullyQualifiedErrorId : UnauthorizedAccess
```
아래 command를 수행하고 policy가 변경되었는지 확인해본다.  
```console
Set-ExecutionPolicy Unrestricted  
  
ExecutionPolicy  
> Unrestricted  
```
***
  