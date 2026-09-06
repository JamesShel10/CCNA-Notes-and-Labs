ACLs - Access Control Lists
#Things we'll cover
1) What are ACLs?
2) ACL logic
3) ACL types
4) configure two kinds of ACLs:
   1) Standard numbered ACLs
   2) Standard named ACLs 


1) What are ACLs?
   - Access Control Lists
   - ACLs function as a packet filter, instructing the router to permit or discard specific traffic.
   - ACLs can filter traffic based on source/destination ip addresses, source/destination Layer 4 ports
     
2) How ACLs work (ACL logic)?
   - ACLs are configured globally on the router (global config mode)
   - Configuring an ACL in global config mode will not make the ACL take effect.
   - They are an ordered sequence of ACEs(Access Control Entries)
   - The ACL must be applied to an interface
   - ACLs are applied either inbound or outbound
     
     
   
   <img width="389" height="125" alt="image" src="https://github.com/user-attachments/assets/02938648-d108-48d4-9684-7766a95f9b3c" />

#Example
Scenario: we will not allow 192.168.2.0/24 network to be able to access SRV1 on 10.0.1.0/24 network.
ACL1:
     A: if source ip = 192.168.1.0/24, then permit
     B: if source ip = 192.168.2.0/24, then deny
     C: if source ip = any, then permit
Requirement:
   - 192.168.1.0/24 can access 10.0.1.0/24
   - 192.168.2.0/24 can't access 10.0.1.0/24

So if you apply outbound ACL through g0/2 interface on R1, it will not take affect because outbound ACL controls traffic leaving an interface (e.g. pc3 with ip 192.168.2.1 send package to it gateway g0/2 - R1 to reach SRV1, between pc3 to R1 that call package(in) and when reach R1 package is (out) through g0/0 to R2 and R2 g0/1 to SRV1) but according to ACL1, we set outbound ACL1 on R1 g0/2 interface it won't work. To say it in harsh word "meaning less ACL configuration".

- To be affect, you need to configure inbound ACL through g0/2 on R1. The R1 will check the ACL1: from top to bottom







   
