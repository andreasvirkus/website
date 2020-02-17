+++
date = 2020-01-16T06:08:00Z
draft = true
title = "Discovering what is Azure Resource Graph"

+++
I recently came across yet another [Azure service](https://azure.microsoft.com/en-us/services/) called [Azure Resource Graph](https://docs.microsoft.com/en-us/azure/governance/resource-graph/overview) and while it seems to be fairly self-explanatory they do make some claims right at the beginning (emphasis mine):

> by providing **efficient** and **performant** \[sic\] resource exploration

But nowhere that I could find do they back this or any of the performance claims up with specific numbers. There some examples online of other people querying their own resources and comparing against straight-up cmdlets, but I wanted something that would give me better data and would allow me to say definitively when I should use what. Though, it must be said that as soon as you're getting into combining multiple calls to resource providers to get the picture you need then you're probably better off turning to Azure Resource Graph queries right away, but let's have some numbers!

## The script

## The examples

## Conclusion

***

Side note: on an editorial tangent I learned that ["performant" isn't a word](https://weblogs.asp.net/jongalloway/performant-isn-t-a-word), which is why I put "[sic]" in the quote.