---
title: Split-ticketing saving amounts
---
How are we using the data from Assertis. Omio is not just a frontend for providers, to integrate all the providers we integrated we need to do some abstraction on top of them. Some providers do not return round-trip responses and we need to create the two-single case ourselves. Because of this we are using only the bundled part of the response and the two-singles part is build on Omio side.

* For Return fare we are asking Assertis
* For Two-singles roundtrip we are asking Assertis for 2 one-ways and then doing the merging on our side
	* More flexible for Omio for example we can implement earlier/later capability on top of the providers
* Given this how can Assertis help us regarding saving amounts?
* In the best case scenario we would somehow get the split-saving data from one-ways, but even in the worst case scenario of us changing the contract completely on our side (taking the savings from round-trip response and mapping it to our two-singles use-case) we have earlier/later implementation, which is transparent to Assertis (seems like normal calls for them) so even if they would give us the saving amounts there is no way for them to know how much should it be given they don't know what else is available in the search on our side.
* Based on this I think getting the data from Assertis would be more effort on our side, with additional costs when we need to change the computations somehow.

Anyway this is the computation logic Per journey:

1. Find minimum offer
2. If it’s not ST => No discount
3. If the offer is same-class
	1. There is no regular offer with the same-class combination => No discount
	1. There is a regular offer with a same-class combination => Subtract
5. If the offer is not same-class
	1. There is no regular offer => No discount
	2. There is a regular offer => Subtract