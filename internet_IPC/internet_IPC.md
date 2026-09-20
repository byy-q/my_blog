# This is the a blog to introduce Internet process communicaion

## socket

socket is an abstraction of communication,it contains 
-protocol family,type,status
-the IP address,the port number
-the send queue,the receive queue
-the timer,the wait queue
-the crowded control,the flow control
-the buffer

so when you create a socket ,you need to specify what Internet layer protocol  you want to use,for instance IP,what transimit protocol you want use,for example ,TCP/UDP.
when you pointed out of these thing ,the OS just know what exact socket you need.

when you need a socket,on unix like system ,you just do this:

    int socket_fd = socket(int domain,int type,int protocol);

so,the POSIX define following domain parameter:
-AF_INET,this refers to IPv4
-AF_INET6,this refers to IPv6
-AF_UNIX,this means you just want to use socket to communicate with the process that locate in the same host
-AF_UPSPEC,which means 

and the type parameter:
-SOCK_DGRAM fixed length,none connection,unreliable transimit,always be UDP
-SOCK_RAW this means you cross the transmit layer,go ahead to IP
-SOCK_SEQPACKET fixed length,ordered,reliable,connection orentation
-SOCK_STREAM ,ordered,reliable,mutual,connection orentated connection,always be TCP

and the protocol parameter
-0,means you choose the default protocol for the domain and the type you chose.for example it is TCP for the SOCK_STREAM,the UDP for SOCK_SGRAM.
-IPPROTO_IPv6
-IPPROTO_TCP
-IPPROTO_UDP
-IPPROTO_RAW
In conclusion,the domain decide the address format,the type decide the feature of the communication,and the protocol decide the concrete protocol.for example,you write this:

    socket(AF_INET,SOCK_RAW,IPPROTO_IPv6)

means you want use IP4 address format,but you actually use the IP6 protocol,this is useful in IP6 and IP4 tunnul.

### Socket address

when you choose AF_INET or AF_INET6,you choose different address format,if you choose AF_INET(IP4),the struct should be:

    struct sockaddr_in
    {
        sa_family_t sa_family;//the address family
        in_port_t   sin_pot;//the port number
        struct in_addr  sin_addr;//the ip address,4 bytes here
    }

when you choose AF_inet6,the struct like this:

    struct aockaddr_in6
    {
        sa_family_t sa_family;//the address family
        in_port_t   sin_port;//the port number;
        uint32_t    sin6_flowinfo;//flow information
        struct in6_addr sin6_addr;//the ip address,16bytes here
        uint32_t    sin6_scope_id;
    }

when you usr socket related function,they all enforced cast to same struct 

    struct sockaddr
    {
        sa_family_t sa_family;
        char    sa_data[14];
    }

the size of the sockaddr_in6 must biger than sockaddr,so when usr related function,you have to pass the length:

    struct sockaddr_in6 sa6;
    bind(fd,(struct sockaddr*)&sa6,sizeof(sa6));

## this partion contain some information consult function
### host name 
on your computer,here are some files,check /etc/hosts,you find pairs of IP address and the domain name;when we usr information function,it might read the hosts file,or it can use DNS service

    struct hostent* gethostent(void);
    void sethostent(int stayopen);
    voidendhostent(void);

first,you should call sethostent() to open the datalib,like the file hosts,and you use the gethostent() to read one entry to the struct hostent,if you use parameter 0,gethostent() will close the file after read.when you end read ,you call endhostent(0) to close.
and the what's struct hostent be like:

    struct hostent
    {
        char*   h_name;//host name
        char**  h_aliases;//other alternative host name
        int     h_address_type;//address tpe,ip4 or ip6
        int     h_length;//the address length in byte
        char**  h_addr_list;//some other ip address for the host,one host might have serval ip address or name
    }
### network name and address
similiar function to get the name and the address:

    struct netent* getnamebyaddr(uint32_t net,int type)
    struct netent* getaddrbyname(const char* name)
    struct netent* getnetent();
    void setnetent(int stayopen);
    void endnetent();

    struct netent
    {
        char*   n_name;//new work name
        char**  n_aliases;//alternative name 
        int     n_addrtype//AF_INET for instance
        uint32_t    n_net;//newwork number
    }

these function is similiar with function of gethostent(),but what's the difference between hostname and network name?
host name is a computer's name,but network name isn't.netwrok name is a net.
### protocol name and number

    struct protoent* getprotobyname(const char* name);
    struct protoent* getprotobynumer(int proto);
    struct protoent* getprotoent();
    void    setprotoent(int stayopen);
    void    end protoent();

you can get protocol name by passing protocol number to getprotobynumber(),also,you can get a number for protocol by passing protocol name to getprotobyname(),the left three function stay the same as said.

### port and the service

    struct servent* getservbyname(const char* name,const char* proto);
    struct servent* getservbyport(int port,const char* proto);
    struct servent* getservent();
    void setservent(int stayopen);
    void endservent*();

    struct servent
    {
        char* s_name;//service name
        char** s_aliases;//pointer to alternate service name array
        int     s_port;//port number
        char*   s_proto;//name of protocol
    }

### getaddrinfo

you can map a host name or service name to a socket address using getaddressinfo().

    int getaddressinfo(const char* restrict host,const char* restrict service
    ,const struct addrinfo*restrict hint,struct addrinfo** restrict res);

the function returns a linklist of struct addrinfo,stored in res.the parameter host stand for the hostname,also service.and the hint is a kind of flliter.the addrinfo contains following member:

    struct addrinfo
    {
        int     ai_flags;//you add flags to customize your consultation
        int     ai_family;//address family,the domain in function socket
        int     sokettype;//the type parameter in function socket
        int     ai_protocol//the protocol type in function socket
        socklen_t     ai_addrlen;//legth in bytes of address
        struct sockaddr*    ai_addr;//address
        char*   ai_connname;//caninical name of host
        struct addrinfo*    next;
    }

when you use hint,you add flags to hint,but the rest member must be 0,and the result you get align to the flags.
after inquiring,free the memory using free func:

    void freeaddrinfo(struct addrinfo* ai);

getnameinfo doing the reverse processof getaddressinfo.

    int getnameinfo(const struct sockaddr* restrict addr,socklen_t alen,

    char* restrict host,socklen_t hostlen,char* restrict service,socklen_t servlen,int flags)

## build connect

if you're develop a client program,you just need to choose correct socket type and protocol,when you call connect(),kelnel will bind your socket to an address automatically.but if you're develop the server program,you need bind your socket to a all_known port mannually.

    int bind(int sockfd,sockaddr* addr,socklen_t* len);

reversely,you can use getsockname() to get an address bind to a socket

    int getsockname(int sockfd,struct sockaddr*restrict addr,socklen_t* restrict len);

also ,if the socket has connect to another socket,you can call getpeername() to get the address of peer()

    int getpeername(int sockfd,struct sockaddr*restrict addr,socklen_t* restrict len);

if you chose SOCK_STREAM or SOCK_SEQPACKET things that connection orentated,you need to build connection before starting send data;

    int connect(int sockfd,const struct sockaddr* addr,socklen_t* len);

when you create a socket ,OS default treat it as a active socket,but the server socket msut be passive,using listen(),transform your socket into passive socket and wait for connection request

    int listen(int sockfd,int backlog);

using accept() to accept the request:

    int accept(int sockfd,struct sockaddr* addr,socklen_t* len);

accept return a file decription(socket fd),this new socket used to communicate with the request socket,the old socket keep waiting for new request.the addr contains the address infomation of the requestor.