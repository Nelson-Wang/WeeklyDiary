“小白”成长记—GPON技术基础知识学习总结
日期：2019-06-14 18:10浏览：459评论：0
GPON技术基础知识学习总结

1.      GPON概念及参考模型

PON（Passive Optical Network）是一种点到多点（P2MP）结构的无源光网络，而GPON（Gigabit Passive Optical Network）是PON（Passive Optical Network）技术中的一种千兆比特PON，采用单根光纤将OLT、分光器和ONU连接起来，上下行采用不同的波长进行数据承载，上行采用TDMA方式上传数据，下行采用广播方式发送数据。GPON支持7种不同的上下行速率组合方式，其中最常使用的是方式是2.4Gbit/s的下行速率以及1.2Gbit/s的上行速率。GPON网络模型可参考下图：



图1.1 GPON网络模型

关于上述网络模型中涉及到的缩略词解释如下：

ONU（Optical Network Unit）：光网络单元，是位于客户端的给用户提供各种接口的用户侧单元或终端，OLT和ONU通过中间的无源光网络ODN连接起来进行互相通信。

ODN（Optical Distribution Network）：光分配网络，是由光纤、一个或多个无源分光器等无源光器件组成（免维护），在OLT和ONU间提供光通道，起着连接OLT和ONU的作用，具有很高的可靠性。

OLT（Optical Line Terminal）：光线路终端是放置在局端的终结PON协议的汇聚设备。

IFgpon: GPON Interface：GPON接口。

CPE（Customer Premises Equipment）：用户侧设备，位于终端用户驻地的设备，通常是电话或其它服务设备。

SNI（Service Node Interface）：业务节点接口。

UNI（User Network Interface）：用户网络接口。

GPON主要特点：支持Triple-play，支持高带宽传输，支持长距离接入（接入层最大半径可达60KM）以及分光特性（1：128）。

2.      GPON技术涉及到的基本概念

2.1 GEM帧

GEM（GPON Encapsulation Mode）帧是GPON技术中最小的业务承载单元，最基本的数据结构。通过唯一的GEM Port ID标识，OLT进行全局分配，OLT和ONU之间的业务虚通道。

在GEM port中绑定某种类型的T-CONT，实现某种业务的QoS控制。比如，承载E1业务的GEM port需要绑定Type1（Fixed bandwidth）的T-CONT。

2.2 T-CONT

T-CONT（Transmission Container）是GPON上行方向承载业务的载体，所有的GEM Port都要映射到T-CONT中，如下图2.1。是上行带宽最基本的控制单元，也是DBA实现的基础。



图2.1 GEM Port与T-CONT的映射关系

每个T-CONT由Alloc-ID来唯一标识，Alloc-ID的范围为0-4095，Alloc-ID由OLT进行全局分配，即OLT下的每个ONU/ONT不能使用Alloc-ID重复的T-CONT。每个T-CONT由一个或者多个Port-ID组成。Port-ID标识的是OLT和ONU/ONT之间的业务虚通道，即承载业务流的通道，类似于ATM虚连接中的VPI/VCI标识，每个Port-ID承载一个业务流，一个T-CONT只能承载一种业务类型的数据流。

T-CONT的带宽类型分为：

（1）FB（fix bandwidth）：固定带宽，不管tcont有没有数据要上传，都分配固定值的带宽，并一直预留。 固定带宽是有无数据传送都占用带宽。

（2）AB（assure bandwidth）：保证带宽，一旦tcont有数据要传，这个带宽就必须预留，不管传的数据有多少，都预留一定带宽。保证带宽为有数据要传的时才去占用带宽。

（3）NAB（non-assure bandwidth）：非保证带宽，fix和assure占用完后，线路中若还有一定的带宽可以利用，那么这些剩下的带宽就优先给非保证带宽利用。

（4）BE（best-effort bandwidth）：最大努力带宽，在fix、assure、non-assure都已经占用完成后，仍然有空间，那么这个空间就留给了be最大努力带宽。

T-CONT共包含五种不同的类型：即type1，type2，type3，type4和type5，如下图2.2所示。type1就是fix带宽，type2就是assure，而type3是一部分asure，一部分non-assure，type4是best-effort，而type5就是所有这4种类型fix、assure、non-assure、best-effort的组合。



图2.2 T-CONT五种类型

3.      GPON协议栈

GPON系统协议栈主要由物理媒质相关（PMD）层和GPON传输汇聚（GTC）层组成。



图3.1 GPON协议栈

3.1 PMD层

GPON的PMD层对应OLT和ONU之间的GPON接口，具体参数值决定了GPON系统的最大传输距离和最大分光比。

3.2 GTC层

GTC层封装ATM信元和GEM帧两种格式的净荷，通常GPON系统采用GEM帧封装模式。GEM帧可以承载以太、POTS、E1、T1多种格式的信元。

GTC层是GPON的核心层，主要完成上行业务流的媒质接入控制和ONU注册。以太帧净荷或者其他内容封装在GEM帧中，打包成GTC帧，按照物理层定义的接口参数转换为物理01码进行传输，在接收端按照相反的过程进行解封装，接收GTC帧，取出GEM帧，最终把以太净荷或者其他封装的内容取出以达到传输数据的目的。

3.2.1 GTC层按结构划分

GTC层按结构可分为GTC成帧子层和TC适配子层。

在TC适配子层，包括ATM适配器、GEM TC适配器和OMCI适配器。ATM适配器、GEM TC适配器通过VPI/VCI或者GEM Port ID识别OMCI通道。OMCI适配器可以和ATM适配器、GEM TC适配器交换OMCI通道数据并传送到OMCI实体上。此外，DBA控制模块为通用功能模块，负责完成ONU报告和所有的DBA控制功能。

在GTC成帧子层，GTC帧可分为GEM块、PLOAM块和嵌入式OAM。GPON成帧子层包括三个功能：

（1）复用和解复用：PLOAM和GEM部分根据帧头指示的边界信息复用到下行TC帧中，并可以根据帧头指示从上行TC帧中提取出PLOAM和GEM部分。

（2）帧头生成和解码：下行帧的TC帧头按照格式要求生成，上行帧的帧头会被解码。同时直接封装在GTC帧头的嵌入式OAM信息被终结，并用于直接控制该子层。

（3）基于Alloc-ID的内部路由功能：根据Alloc-ID的内部表示为来自或者送往GEM TC适配器的数据进行路由。

3.2.1 GTC层按功能划分

GTC层按功能可分为C/M平面和U平面。

C/M控制管理平面的协议栈包括三部分：嵌入式OAM，PLOAM（物理层OAM），OMCI（ONT管理控制接口）。嵌入式OAM和PLOAM通道负责管理PMD和GTC子层的功能，OMCI为更高子层管理提供统一的系统。嵌入式OAM通道位于GTC帧的帧头，主要功能包括：带宽确认、交换、动态带宽分配信令等。PLOAM通道是一个格式化的消息系统，在GTC帧中占据专有空间，主要用于承载那些不通过嵌入式OAM发送的PMD和GTC管理信息。OMCI通道用于管理业务。

U平面内的业务流用业务流类型（ATM、GEM）及其端口ID或VPI来标识。端口ID用于识别GEM业务流，VPI用于识别ATM业务流。在T-CONT中通过可变的时隙控制来实现带宽分配和QoS控制。

4.      GPON关键技术

4.1 测距

ONU到OLT的距离各自不同，在ONU以TDMA方式（也就是在同一时刻，OLT一个PON口下的所有ONU中只有一个ONU在发送数据）发送上行信元时可能会出现碰撞冲突。为了保证各ONU的上行突发不冲突，需要使用同一个时间基准。

测距实现过程主要如下：

（1）OLT在ONU第一次注册时就会启动测距功能，获取ONU的往返延迟RTD（Round Trip Delay），计算出每个ONU的物理距离。

（2）根据ONU的物理距离指定合适的均衡延时参数（EqD：Equalization Delay）。

此过程相当于把ONU的RTD（Round Trip Delay：环路时延）拉长到一个相同的标准值，即所有ONU都在同一逻辑距离上，在对应的时隙发送数据即可，从而避免上行信元发生碰撞冲突。

4.2 突发光电技术

GPON上行方向采用时分复用的方式工作，每个ONU必须在许可的时隙才能发送数据，不属于自己的时隙必须瞬间关闭光模块的发送信号，才不会影响其他ONU的正常工作。对于OLT侧上行接收来讲，必须要根据时隙进行突发接收每个ONU的上行数据，因此，为了保证GPON系统的正常工作，ONU侧的光模块除连续发送模块外必须有突发发送模块（如下图4.1），OLT侧的光模块除连续接收模块外必须有突发接收模块（如下图4.2）。

Ps：GPON下行是按照广播的方式将所有数据发送到ONU侧，因此，要求OLT侧的光模块必须连续发光，ONU侧的光模块也是连续接收方式工作，所以在GPON下行方向，OLT光模块无需具有突发发送功能，ONU光模块无需具有突发接收功能。



图4.1 ONU侧连续发送模块和突发发送模块示意图



图4.2 OLT侧连续接收模块和突发接收模块示意图

4.3 DBA

DBA(Dynamically Bandwidth Assignment)：动态带宽分配，完成对PON线路上行带宽进行动态分配的一种机制。根据ONU/ONT突发流量需要，通过在ONU/ONT之间动态调整带宽提高了PON上行带宽的有效性，提高PON端口的上行线路带宽利用率，为PON口上增加更多的用户。

动态分配带宽只对额外带宽进行分配。按T-CONT的保证带宽比例，从剩余带宽中分配非保证带宽给要求额外带宽的每个T-CONT。如果保证带宽是0，则要公平地分配非保证带宽和最大努力带宽给每个T-CONT。在带宽分配中，首先分配固定带宽；然后，分配保证带宽；最后把仍然没有被预约的带宽放到剩余带宽库中，作为非保证带宽和最大努力带宽。

4.4 QoS

QoS：Quality of Service（服务质量）是指网络通信过程中，允许用户业务在丢包率、延迟、抖动和带宽等方面获得可预期的服务水平。它是网络的一种安全机制， 是用来解决网络延迟和阻塞等问题的一种技术。

在正常情况下，如果网络只用于特定的无时间限制的应用系统，并不需要QoS，比如Web应用等。但是对关键应用和多媒体应用就十分必要。当网络过载或拥塞时，QoS 能确保重要业务量不受延迟或丢弃，同时保证网络的高效运行。如VOIP业务，无论网络多繁忙，都要保证其带宽及时延。

QoS具有以下作用：

为用户提供带宽保证；减少报文的丢失率；避免和管理网络拥塞；调控IP网络的流量；设置报文的优先级；为不同的用户业务提供差异化服务。

5.      应用场景

GPON采用无源光传输技术，主要应用于FTTx解决方案，包括：FTTM（Fiber To The Mobility Base Station）、FTTO（Fiber To The Office）、FTTB（Fiber To The Building）、FTTC（Fiber To The Curb）、FTTD（Fiber to The Door）、FTTW（Fiber To The WLAN）、FTTH（Fiber To The Home）和D-CCAP（Distributed-Converged Cable Access Platform）组网场景，支持语音、数据、视频、专线接入和基站接入业务。

（1）FTTH：OLT通过ODN与用户家里的ONT连接。FTTH组网适合相对分散的新建高档公寓或别墅，为高端用户提供更高带宽的服务。

（2）FTTB/FTTC：OLT通过ODN与楼道（FTTB）或路边（FTTC）的ONU连接，ONU与终端用户相连。FTTB/FTTC组网适合用户比较集中的小区或写字楼，为普通用户提供一定带宽的服务。

（3）FTTO：OLT通过ODN与企业的ONU连接，ONU与终端用户相连。FTTO组网适合企业网，实现企业内TDM PBX、IP PBX，及企业内部网的专线业务。

（4）FTTM：OLT通过ODN与ONU相连，ONU与无线基站相连。FTTM组网适合移动承载网改和扩容，实现承载层面的固定网络与移动网络的融合。

（5）FTTW：OLT通过ODN与ONU相连，ONU与AP相连，实现WLAN流量回传，成为Wi-Fi建设的未来趋势。

（6）D-CCAP：D-CCAP建设场景，采用统一PON接入平台、分布式部署方式，可满足家庭、企业、热点覆盖的需求，是HFC网络承载三网融合业务的新引擎。

GPON接入方式下FTTx组网的共同点是：终端用户的数据、语音、视频信号经过ONU后被转换为以太报文，由ONU的GPON上行端口通过光纤传输到OLT，然后从OLT上行端口转发至上层IP网络。

总结：

文章可以看做是一篇综述，看了很多篇关于GPON技术的介绍后，将其中涉及到的基本概念以及关键技术和应用场景做了简单的概括。应该是入门GPON首先要了解的基础知识，整理一遍基础知识能够加深印象，并有一个系统的梳理过程。

 
