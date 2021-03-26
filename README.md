# lithrion.github.io

This is a portfolio of cybersecurity related projects that I have done. You can get back to the webpage view [here](https://lithrion.github.io).

This portfolio was designed using Jekyll, markdown, and a little bit of liquid.

In this portfolio you will find:

# Class Projects
I took a cybersecurity bootcamp offered through UCSD extension and have included a few of the class projects. This includes an example of deploying and ELK stack which was tested on an Azure virtual network. There is also a final projects which demonstrates both offensive and defensive sides of cybersecurity.

# Hack the Box Walkthroughs
To practice offensive security techniques, I've begun working on breaking into some machines hosted by Hack the Box. I'll be posting writeups of the machines here. After they're retired of course to avoid spoilers.

# Other Projects
Working on an ecommerce store, I built a bash script to automate generating product pages from a csv file of product specs.

# Current Projects
Currenly I'm looking into learning ruby on rails. What I would like to do, is automate the process of printing out new barcodes when products get added into an ecommerce store. The problem that I ran into, is that the webhook coming from the ecommerce webserver expects HMAC validation to be performed and a response code sent back. The barcode software expects login credentials rather than a digital signature.

My planned solution is to use ruby on rails to build a relatively simple web app that receives and verifies the webhook, and then forwards the product data on to the barcode software. The issue that I've run into is that the tutorials for the ecommerce platform, as well as the command line tools for generating boilerplate code, appear to be based off outdated versions of the API that either don't function as intended or don't function at all.

If they had been working, I think the material from the tutorials would have been able to accomplish my goal. Since they aren't, I'll have to delve into ruby a little deeper to find a solution.
