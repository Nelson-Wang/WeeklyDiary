一、ONT上线分为以下步骤
1、配置DBA模板

2、配置线路模板：创建线路模板（ont-lineprofile）、建立TCONT并映射到DBA（tcont）、建立GEMpoet并映射到TCONT（gem add）、建立GEMport与ONT侧业务的映射使得gem可以通过vlan或指定端口转发（gem mapping）

3、配置业务模板：创建业务模板（ont-srvprofile）、配置ONT以太端口的能力集（ont-port eth）、设置ONT端口vlan（port vlan eth）

4、ONT上线：使能ONT自动查找（port 0 ont-auto-find enable）、端口ONT确认（ont confirm 0）


二、ONT上线过程
1、确保接线成功，记录下PON板在OLT上的槽位，PON上的端口号。比如我的环境下PON板在6号槽位上，端口是0。
通过display board 0可以看到OLT板子的状态。如图，我的PON板接在6号槽上，OLT自动找到（Auto_find）了该PON板。
 
2、通过board confirm 0/6确认自动发现的单板（注意这里0/6因为我的PON板在6号槽上）。如图，提示成功确认 。

这时候可以通过再次查看board 0（display board 0）的信息看PON是否却确认成功。
 
3、配置DBA模板（DBA就是对ONT上行数据传输进行控制）
通过display dba-profile可以查看当前的DBA模板（DBA有5种类型）
 
可以直接用当前的模板，也可以自己创建。下面演示自己创建一个DBA模板：dba-profile add 
 
执行上述命令之后，我们得到了模板id为12、类型为type1、固定字节10M、模板名字为mjs1的DBA模板。
 
4、配置线路模板（可以理解为ONT到OLT之间的通路）
	a、创建线路模板：执行ont-lineprofile gpon profile-id 10 profile-name mjs2
	b、TCONT绑定一个DBA模板，用于实现动态带宽分配：（将TCONTID为2绑定到id为12的DBA模板上）
              tcont 2 dba-profile-id 12            
	c、配置ONT线路模板中GEM（GPON Encapsulation Mode）Index与T-CONT的绑定关系及相关属性，建立承载业务的GEM Port：
             gem add 4 eth tcont 2（将gem port 1绑定到tcont2上）
	d、配置GEM Mapping，建立GEM Port与ONT用户接口数据流的映射关系，使得gem可以通过指定的vlan或端口转发：
              gem mapping 1 0 vlan 100（指定只有vlan为100的数据才能通过）
              gem mapping 0 0 eth 4（所有数据都可以通过ONT以太口4透传）
              mapping-mode port（切换映射关系为端口） 

线路模板配置完成后记得  commit！！！（不commit前面的配置就没了）
 
5、配置ONT业务模板（承载ONT侧的业务流，数据从ONT的哪个口进，哪些数据可以进。同时指定ont的端口数）
	a、创建业务模板：执行ont-srvprofile gpon profile-id 1 profile-name mjs_srv1
 	b、配置ont-ports 的能力集，就是它实际支持的物理接口:
              ont-port eth adaptive pots adaptive(自适应配置物理端口数）
              ont-port eth 4（配置物理端口数为4）
	c、配置端口vlan，让vlan为100的数据可以通过猫上面的eth口4传进来,实验组网中ont连接的是第几个以太口就用几号，本实验插的是4号口
             port vlan eth 4 transparent：指定1口为透传！！！所有vlan都可以进入ONT。
             port vlan eth 4 translation 101 user-vlan 100：以太4口将vlan为101转化为vlan为100发送出去。只有vlan是101才能从4口传输！
             port vlan eth 4 100：将以太口4的vlanid设置为100）  
commit后quit
6、ONT上线
	通过interface gpon命令进入对应的pon板槽位号，然后进行端口自动查找ONT使能：port 0 ont-auto-find enable（使能端口0自动发现ONT)。
	接着ont confirm 0对端口进行ont确认。
再display ont info 0 all查看猫是否上线。如图，表示端口下有三个ONT，并且都已经上线。
 
