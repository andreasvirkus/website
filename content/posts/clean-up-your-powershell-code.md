---
draft: true
title: Clean up your PowerShell code
summary: 'Let it be known that I am stickler for clean and readable code, and with that come many moments of yak shaving before I feel good about the "final" version of the code. These are some of the methods I have come across that can help.'
---

Let it be known that I am stickler for clean and readable code, and with that come many moments of yak shaving before I feel good about the "final" version of the code. These are some of the methods I have come across that can help.

## Splat calculated properties

A lot of people might be familiar with regular splatting, which allows us to clean up long lines (and they can get _really_ long) such as this:

{{< highlight powershell >}}
Get-AzLog -ResourceGroupName 'rg-eed5657e-c2ca-410f-8aff-f839492ba7b6' -StartTime '2019-12-27T10:30' -EndTime '2019-12-27T11:30' -Verbose
{{< / highlight >}}

to something like this:

{{< highlight powershell >}}
$AzLogParams = @{
    ResourceGroupName = 'rg-eed5657e-c2ca-410f-8aff-f839492ba7b6'
    StartTime = '2019-12-27T10:30'
    EndTime = '2019-12-27T11:30'
    Verbose = $true
}

Get-AzLog @AzLogParams
{{< / highlight >}}

And this could be cleaned up even further by using well-named variables, but when you now need to parse the output of this command and get one of its many properties, then you will run into an issue where almost all of the useful properties need additional calculations. This is where splatting calculated properties comes in!

{{< highlight powershell >}}
Get-AzLog -ResourceGroupName 'rg-eed5657e-c2ca-410f-8aff-f839492ba7b6' -StartTime (Get-Date).AddDays(-1) | Select-Object -Property @{n='EventTimeStamp'; e={ Get-Date -Date ($_.EventTimeStamp) -Format 's' } }, @{n='Operation'; e={$Result = $_.OperationName.value -split '/'; $Result[-2], $Result[-1] -join ' - '}}, @{n='Resource'; e={($_.ResourceId -split '/')[-1]}}, @{n='Status'; e={$_.Status.value}}, @{n='SubStatus'; e={$_.SubStatus.LocalizedValue}}
{{< / highlight >}}

I think we can all agree that the command above is far too long and requires far too much effort to grok. You could just shorten the first bit by splatting the parameters relevant to the `Get-AzLog` cmdlet, but that would still leave you with a very long line:

{{< highlight powershell >}}
$AzLogParams = @{
    ResourceGroupName = 'rg-eed5657e-c2ca-410f-8aff-f839492ba7b6'
    StartTime = (Get-Date).AddDays(-1)
}

Get-AzLog @AzLogParams | Select-Object -Property @{n='EventTimeStamp'; e={ Get-Date -Date ($_.EventTimeStamp) -Format 's' } }, @{n='Operation'; e={$Result = $_.OperationName.value -split '/'; $Result[-2], $Result[-1] -join ' - '}}, @{n='Resource'; e={($_.ResourceId -split '/')[-1]}}, @{n='Status'; e={$_.Status.value}}, @{n='SubStatus'; e={$_.SubStatus.LocalizedValue}}
{{< / highlight >}}

You could shorten it even more by splitting the lines after each comma, which would take you a step in the right direction:

{{< highlight powershell >}}
Get-AzLog @AzLogParams | Select-Object -Property @{n='EventTimeStamp'; e={ Get-Date -Date ($_.EventTimeStamp) -Format 's' } },
@{n='Operation'; e={$Result = $_.OperationName.value -split '/'; $Result[-2], $Result[-1] -join ' - '}},
@{n='Resource'; e={($_.ResourceId -split '/')[-1]}},
@{n='Status'; e={$_.Status.value}},
@{n='SubStatus'; e={$_.SubStatus.LocalizedValue}}
{{< / highlight >}}

but is a tad short from what _I_ see to be the superior solution:

{{< highlight powershell >}}
$AzLogParams = @{
    ResourceGroupName = $ResourceGroupName
    StartTime = $StartTime
}

$SelectObjectParams = @{
    Property = @{ Name = 'EventTimeStamp';
               Expression = { Get-Date -Date ($_.EventTimeStamp) -Format 's' } },
               @{ Name = 'Event';
               Expression = { $_.EventName.value } },
               @{ Name = 'Operation';
               Expression = { ($_.OperationName.value -split '/')[-2] } },
               @{ Name = 'ResourceName';
               Expression = { ($_.ResourceId -split '/')[-1] } },
               @{ Name = 'Status';
               Expression = { $_.Status.value } },
               @{ Name = 'SubStatus';
               Expression = { $_.SubStatus.LocalizedValue } }
}

Get-AzLog @AzLogParams | Select-Object @SelectObjectParams
{{< / highlight >}}
