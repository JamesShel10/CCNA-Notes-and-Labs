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
   - ACLs are defined in global config but must be applied to an interface to take effect.
   - ACLs are applied either inbound or outbound
   - They are an ordered sequence of ACEs(Access Control Entries) and made up of one or more ACEs.
   - Note/ order of ACL is important
   
     
   
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

So if you apply outbound ACL through g0/2 interface on R1, it will not take effect because outbound ACL controls traffic leaving an interface (e.g. pc3 with ip 192.168.2.1 send package to it gateway g0/2 - R1 to reach SRV1, between pc3 to R1 that call packet(in) and when reach R1 packet is (out) through g0/0 to R2 and R2 g0/1 to SRV1) but according to ACL1, we set outbound ACL1 on R1 g0/2 interface it won't work. To say it in harsh word "meaning less ACL configuration".

- To be affect, you need to configure inbound ACL through g0/2 on R1. The R1 will check the ACL1: from top to bottom

Note/ if you set inbound ACls on g0/2 interface, pc from those network will not be able to access not only SRV1 but also other traffic network too. so that's not good neither. 

#One logic thinking: What will be the best way to set ACLs on which interface?

#Implicit deny
- Implicit deny ocure when a packet doesn't match any of the entries in an ACL.

e.g. ACL2:
      - 1) if source Ip = 192.168.1.0/24 then permit
      - 2) if source IP = 192.168.0.0/16 then deny
      - 3) if source Ip = any, then deny

      but the source ad destination is: 10.0.0.1 and 1.1.1.1 so the packet doesn't match to the apply ACL.


3) ACL Types
   1) Standard ACLs:
      - match traffic based on Source IP address (only)
      #Two types of standard ACLs:
         1) Standard Numbered ACLs
            - identified with a number (ie. ACL1, ACL2 )
            - different types of ACLs have a different range of numbers that can be used.
                 (Standard ACLs can use 1 - 99 and 1300 - 1999)
            #How to configure Standard numbered ACL
            - R1(config)# access-list number {deny | permit} IP wildcard-mask
              #e.g.
            - access-list 1 deny 1.1.1.1 0.0.0.0
              #or
            - access-list 1 deny 1.1.1.1
            - access-list 1 permit any
              #remark
            - access-list 1 remark ## BLOCK BOB FROM ACCOUNTING DEP##
            #Apply ACL to an interface:
            - R1(config-if)# ip access-group number {in | out}
            #Note:
            - standard ACL should be applied as close to the destination as possible
            #to check standard named ACL configure result:
            - show access-list
            - show running-config | include access-list
               
         2) Standard Named ACLs
            - identified with a name (ie. 'BLOCK_BOB')
            #How to configure standard named ACL
            - R1(config)# ip access-list standard acl-name
            - R1(config-std-nacl)# [entry-number] {deny | permit} ip wildcard-mask
            #e.g.
            - R1(config)#ip access-list standard BLOCK_BOB
            - R1(config-std-nacl)#5 deny 1.1.1.1
            - R1(config-std-nacl)#10 permit any
            - R1(config-std-nacl)# remark ## CONFIGURE NOV 21 2020 ##
            - R1(config-std-nacl)#int g0/0
            - R1(config)#ip access-group BLOCK_BOB in
            #to check standard numbered ACL configure result:
            - show access-list
            - show running-config | section access-list
       
   2) Extended ACLs:
      - match traffic based on Source/Destination IP, Source/Destination port
        #Two types of Extended ACLS:
          1) Extended Numbered ACLs
          2) Extended Named ACLs
        









   
