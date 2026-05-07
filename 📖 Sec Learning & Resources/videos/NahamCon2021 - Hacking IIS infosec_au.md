
<iframe width="560" height="315" src="https://www.youtube.com/embed/cqM-MdPkaWo?si=9TpxwFDeiuKEI9TN" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

tags: #infosec_au  #iis #windows

it started by a tweet he shared of why he love hacking iis

- case insensitive , amazing for content directory
- iis shortname
- viewstate deserialization rce gadgets
- web.config upload tricks
- debug mode W/ detailed stack traces and full path
- debuging scripts often deployed (elmah, trace)
- telirik rce

in iis you should vhost enumeration /bruteforce to see if any other application are present on the host

if you can read web.config  you always almost get rce impact

https://github.com/0xacb/viewgen

when hacking iis server use dnspy  to find vulnerabilty in source code

dnspy is capable of reversing  dll files jus load the dll files in the project

