# ns-pair-checking
Checks each zone file entry in named.zones slave file for non-syntactical errors (written in bash)

---------------------------------------------------------------------------------------------------
This is the Formula for working out exactly how many instances of a domain name wil be in the file.
---------------------------------------------------------------------------------------------------

READ named.zones| cut spaces| count lines

cat named.zones| sed '/^$/d'| wc -l


Divide answer by 5 (5 lines per zone)

Multiply answer by 2 (2 domain names in each instance)


So, in an example with 4 working zone files:

cat file.txt| sed '/^$/d' | wc -l
20

20 / 5 = 4
4 * 2 =  8

There will be 8 instances of a "legitimate" domain name in each example.


I could make this automatic. So I will :')

#---------------------------------------------------------------------------------------------------
# This is my automagicafying scriptamabob
#---------------------------------------------------------------------------------------------------

linesT=$(cat named.zones| sed '/^$/d'| wc -l)
zonesT=$total/5
domsT=$zonesT*2

printf "There will be exactly %n domains in the file.\n" "$domsT"
#---------------------------------------------------------------------------------------------------

Then, I'm going to initialise an array the size of the number obtained from step 1

domainCount[8], in the instance of a 4 domain named.zones file.

The thinking being, the script will read the file line-by-line, adding each domain name it finds to the array.

-----------------------------

grep -o '".*"'| sed 's/"//g'

-----------------------------

Greps out EVERYTHING inbetween double quotes.
Fortunately, the named.zones file looks like this:

zone "domain.com" IN {
        type slave;
        master {IP;};
        file "domain.zone";
};


So, only domain entries will be picked up.


The next thing I need to work out is how I'm going to check each element of the array against itself.

Thinking of using a bubble-sort-like formula. Regardless of whether all the entries are correct or not - the final number should be equal.

The script should (on an individual zone-by-zone basis)-
        Read line 1 - find "domain.com" - add it to position 1 (or 0 dependent on bash array syntax)
        Read line 2 - ignore it
        Read line 3 - ignore it
        Read line 4 - find "domain.com.zone" - add it to position 2 (or 1)

        Skip the }


Again using the example of 4 domains in the zone, there will be 8 domains in an array, when echo'd it should look something like this:

        array[0]=domain.com
        array[1]=domain.com

        array[2]=domain2.com
        array[3]=domain2.com

        array[4]=domain3.com
        array[5]=domain3.com

        array[6]=domain4.com
        array[7]=domain4.com

Notice the pairs of entries. The next thing I want to do, is to remove any domains which don't have a pair. This would insinuate that the entry is wrong.
So say, for example, domain4 was wrong... it might look something like this:

        array[6]=domain4.com
        array[7]=ddomain4.com

Of course the error doesn't have to be the exact same. The actual inputted error does not make a fucking difference.
My objective is to use some sed or grep magic to obtain a list of domains that only appear once.

I'm going to have to programatically split the array into checkable pairs.
I want the script to fill a secondary array with any irregularities- I.e.e

I.e (not 100% syntactically accurate YET)

        if [array[0] != array[1]];then
           array2[0]==array[0];
        fi

Something like that. BUT, I want to use the modulus to split each array element into two categories: odd or even.

This is going to have to be done using constants, modulus and a case-by-case comparison as to whether the element's position in the array is even or odd.

The way I see it working ( theoretically ) :
[again, not syntactically correct]

##Two constants, one and 2
         ODD=1
        EVEN=2

        if [array[x]%2 == ODD];then
           array2[x]=array[x]
        fi

OR SOMETHING LIKE THAT.

I'm thinking of making the secondary array print whether it was the first instance or the second.

That might be over engineering a little bit...

The end goal is simple.

I want a file containing a list of domains from the named.zones file which don't appear twice.

This can be used for debugging any and all non syntactical errors.


If I incorporate the modulus feature, it will give a number/word saying:

        1 or 2
        OR
        Odd or Even.

This would indicate which line (1st or 2nd) the domain spelling is wrong.

Again, this wouldn't pick up syntactical errors... But the theory being, rndc wont reload if the syntax is wrong.

The script could evolve to be an autofix. It could potentially fix the errors for me.
But, that's a long way away. The main goal for now is to ascertain a list of spelling mistakes.




