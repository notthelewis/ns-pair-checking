---------------------------------------------------------------------------------------------------
What it's for
---------------------------------------------------------------------------------------------------


I wrote this script to start fixing problems with one of the nameserver slaves for an older platform. It hasn't (and never could) 
fix the entire nameserver. It's very broken. 


---------------------------------------------------------------------------------------------------
Why it's needed
---------------------------------------------------------------------------------------------------


The script is needed because, as mentioned, one of the nameserver slaves was broken. How it's broken is not important. 

For now, all that's needed is this- Newly registered domains aren't automatically adedd to the named.zones file. 

Therefore, if a customer registers a new domain, puts all their content on and dns is right- it just will not work. 

We're trained to simply add our own entry to the named.zones file. This leaves room for human error... Also weird machine error.

People (or the slave in the process of breaking) were/are putting entries with spelling mistakes.

---------------------------------------------------------------------------------------------------
What the script does
---------------------------------------------------------------------------------------------------

    Prelude
    
The format which this particular file employs is very simple, and easy to grasp. I'd hedge a bet that you could
(as a programmer) know what this does:

        zone "domain.com" IN {
                type slave;      
                master {IP;};       
                file "domain.zone";  
        };

Each domain that is registered on this system, uses that exact format. There's thousands of domains on the system. 

If you can't guess what it does: 
        
Someone types in domain.com, which is hosted at ns.thisCompany.com

Requesting client (browser, email client etc) then goes to ns.thisCompany.com

ns.thisComany.com then searches the file: named.zones for domain.com.

It finds something akin to line 32 (the zone file above)

Then, that file tells it the IP of the server the information is found in (IP is withdrawn for obvious reasons)

And goes to the file that is specified under the file paramater: 

    file "domain.zone";
    
In here it will find the zone file for domain. Make sense? Good. 

Now for the script.

First, I'm going to use some text manipulation techniques to acheive the following (on a per instance basis): 

    Get everything inside double quotes: ""
    Get rid of the ""
    remove .zone at the end of the second instance
    Outputs it to a file called cleanzones

This is described in: 1-Cleaning the files

Now, each domain will appear two times in the cleanzones file. Each zone instance is 5 lines long.

The first command run is to remove the spaces and count how many lines are there in total.

    cat named.zones| sed '/^$/d'| wc -l

Then, I want to divide answer by 5 (5 lines per zone)

And finally, multiply answer by 2 (2 domain names in each instance)


So, in an example with 4 working zone files:


cat cleanzones| sed '/^$/d' | wc -l

20

20 / 5 = 4


4 * 2 =  8


There will be 8 instances of a "legitimate" domain name in each example.


Then, I'm going to initialise an array the size of the number obtained.

domainCount[8], in the instance of a 4 domain named.zones file. Each array element is a "clean" domain name.

Again using the example of 4 domains in the zone, there will be 8 domains in an array

        array[0]=domain.com
        array[1]=domain.com

        array[2]=domain2.com
        array[3]=domain2.com

        array[4]=domain3.com
        array[5]=domain3.com

        array[6]=domain4.com
        array[7]=domain4.com

So say, for example, domain4.com was entered into the file wrongly... it might look something like this:

        array[6]=domain4.com
        array[7]=ddomain4.com
        
This next bit is my favourite part of the script. This compares each element with its partner.

The caveat with that, of course, is if you just check each element with the one next to it, there's going to be a lot of error.

Why?

Go back to the visualisation of the array elements (line 104-line 114)

If every single element is checked against the next in the list, then domain will be compared with domain2...
Domain2 will be compared with domain3 and so on and so forth.

How do we stop this? 

One word... Modulus.

    for (( start = 0; start <= $domsT; start++ ))
    do
      if [ $(($start %2)) == 0 ] && [ "${lines[$start]}" != "${lines[$start+1]}" ]; then
        printf "This line is wrong: ${lines[$start]}\n"
      fi
    done

For each iteration, if the iteration counter can be evenly divided by 2 & the domain doesn't equal the one next to it,
print the element which is causing problems. 

The end result is a list printed to the console of any domains which don't have a properly written zone file. 

It worked :)
