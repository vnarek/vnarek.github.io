---
title: Solutions for booking fragment in API V3
draft: "true"
---

Booking fragments negatively impacted our SLAs and created issues that resulted in missing transactions, corrupted bookings, etc. Let's see if there exists a different solution for our situation.

The reason the booking fragment in Couchbase exists is because some of the fields were missing before in our internal teams, and were not available in both search and booking responses. Additionally, it gave us more flexibility in defining the responses in a way that is more suited to our partners (UBER currently).

# Description of the current solution

Currently, we store some information in the booking fragment (mostly from the search context) and some of the information is additionally fetched from the booking endpoints (tickets representation, booking API).

The solution was brittle before, but after Alex made some changes we did not have any additional issues (except sometimes I forgot to put something into some structure, and it was missing in the fragment).

But the current code is messy, the response time for booking-get is currently at P50 192ms and without the booking fragment, it will be even slower.

# What is currently missing in the booking responses

Probably the most important pieces are the
* Positions
* Carriers
* Providers
because all the other parts of the response refer to these arrays by their IDs. And the second thing is links, which probably contain information about how to create a search.

## Complete search context annotated
```json
{
    "status": "BOOKED", //booking-transactionV4$.status
    "totalPriceCents": 14082, //booking-transactionV4$.totalPriceInDisplayCurrency.lowestUnitValue
    "currency": "EUR", //booking-transactionV4$.totalPriceInDisplayCurrency.currency
    "travellers": [
        {
            "name": "Narek Vardanjan", //booking-transactionV4$.passengers[0].name.first + booking-transaction$.passengers[0].name.last
            "age": 30 //booking-transactionV4$.passengers[0].ageInfo.age
        }
    ],
    "partnerId": "uber", //booking-transactionV4$.partnerId
    "bookingCode": "1604", //ticket-reservations$.reservations[latest].bookingReference
    "ticketURL": "/bookings/f1b40b15-46a6-4d78-b36a-b2e464c319c2/tickets", //ticket-reservations$.reservations[latest]ticketRepresentations
    "reservationValidUntil": "2024-01-04T15:37:18.126503681Z", //booking-transactionV4$.reservationValidUntil
    "searchContext": {
        "searchOptions": {
            "departurePositionId": "380553", //booking-transactionV4$.searchOptions.departurePositionId
            "arrivalPositionId": "380575", //booking-transactionV4$.searchOptions.arrivalPositionId
            "travelModes": [
                "train" //missing
            ],
            "outboundDateTime": "2024-01-05T07:00:00Z", //ticket-reservations$.reservations[latest].segments[0].departureDateTime
            "passengerAges": [
                30 //booking-transactionV4$.searchOptions.passengers[*].ageInfo.age
            ],
            "travellers": [
                {
                    "age": 30 //booking-transactionV4$.searchOptions.passengers[*].ageInfo.age
                }
            ]
        },
        "positions": [ //db-api-v1-nodes-batch$.*
            {
                "id": "318858",
                "type": "station",
                "name": "Deansgate",
                "countryCode": "GB",
                "cityName": "Manchester",
                "lat": 53.4742934,
                "lon": -2.251523
            },
            {
                "id": "319691",
                "type": "station",
                "name": "Manchester Oxford Road",
                "countryCode": "GB",
                "cityName": "Manchester",
                "lat": 53.4739155,
                "lon": -2.2419014
            },
            {
                "id": "380553",
                "type": "location",
                "name": "London",
                "countryCode": "GB",
                "cityName": "London",
                "lat": 51.50853,
                "lon": -0.12574
            },
            {
                "id": "380575",
                "type": "location",
                "name": "Manchester",
                "countryCode": "GB",
                "cityName": "Manchester",
                "lat": 53.48095,
                "lon": -2.23743
            },
            {
                "id": "319631",
                "type": "station",
                "name": "London Euston",
                "countryCode": "GB",
                "cityName": "London",
                "lat": 51.52809902784242,
                "lon": -0.13317672404014
            },
            {
                "id": "319637",
                "type": "station",
                "name": "London Marylebone",
                "countryCode": "GB",
                "cityName": "London",
                "lat": 51.52274372502569,
                "lon": -0.16271176375426
            },
            {
                "id": "319638",
                "type": "station",
                "name": "London Paddington",
                "countryCode": "GB",
                "cityName": "London",
                "lat": 51.5166224,
                "lon": -0.1770154
            },
            {
                "id": "319641",
                "type": "station",
                "name": "London St Pancras International",
                "countryCode": "GB",
                "cityName": "London",
                "lat": 51.53142035092777,
                "lon": -0.12599427434383692
            },
            {
                "id": "319692",
                "type": "station",
                "name": "Manchester Piccadilly",
                "countryCode": "GB",
                "cityName": "Manchester",
                "lat": 53.477376942479914,
                "lon": -2.2309072085288335
            },
            {
                "id": "319694",
                "type": "station",
                "name": "Manchester Victoria",
                "countryCode": "GB",
                "cityName": "Manchester",
                "lat": 53.487577,
                "lon": -2.2423033
            }
        ],
        "providers": [
            {
                "id": "100088", //missing
                "name": "assertis", //ticket-reservations$.reservations[*].provider
                "displayName": "assertis" //ticket-reservations$.reservations[*].provider
            }
        ],
        "carriers": [
            {
                "id": "28", //missing
                "name": "Avanti West Coast", //ticket-reservations$.reservations[*].segments[*].carrierName
                "logoUrl": "https://cdn-goeuro.com/static_content/web/logos/{size}/avanti_west_coast.png" //ticket-reservations$.reservations[*].segments[*].carrierLogoUrl
            }
        ],
        "segments": [
            {
                "id": "1154224989", //ticket-reservations$.reservations[*].segments[*].id
                "travelMode": "train", //ticket-reservations$.reservations[*].segments[*].travelMode
                "departureTime": "2024-01-05T07:13:00Z", //ticket-reservations$.reservations[*].segments[*].departureDateTime
                "arrivalTime": "2024-01-05T09:19:00Z", //ticket-reservations$.reservations[*].segments[*].arrivalDateTime
                "departurePositionId": "319631", //ticket-reservations$.reservations[*].segments[*].departureStationId
                "arrivalPositionId": "319692", //ticket-reservations$.reservations[*].segments[*].arrivalStationId
                "durationMinutes": 126, //ticket-reservations$.reservations[*].segments[*].travelTimeInMinutes
                "carrierId": "28", //missing
                "transportId": "C87733", //ticket-reservations$.reservations[*].segments[*].vehicleId
                "direction": "OUTBOUND", //ticket-reservations$.reservations[*].segments[*].direction
                "noChange": false //ticket-reservations$.reservations[*].segments[*].noChangeForNextSegment
            }
        ],
        "journey": {
            "id": "g6NqaWTOSohQIKNvaWS0LTU5MjE4MjQxOTMyMDY3NDA5NTWjaWlkoA", //missing
            "outbound": {
                "id": "-5921824193206740955", //booking-transactionV4$.journeyInfo.outboundId
                "carrierId": "28", //missing
                "durationMinutes": 126, //sum ticket-reservations$.reservations[*].segments[*].travelTimeInMinutes
                "departureTime": "2024-01-05T07:13:00Z", //ticket-reservations$.reservations[*].segments[*].departureDateTime
                "arrivalTime": "2024-01-05T09:19:00Z", //ticket-reservations$.reservations[*].segments[*].arrivalDateTime
                "segmentIds": [
                    1154224989 //ticket-reservations$.reservations[*].segments[*].id for outbound
                ],
                "travelMode": "train", //ticket-reservations$.reservations[*].segments[*].travelMode
                "routedConnection": false, //missing
                "mobileTicketSupported": false, //can be computed
                "overtakenByJourneyCount": 0 //missing
            },
            "offers": [
                {
                    "id": "R::0edc89fc-5116-4724-83c8-c9c8b5972708::d1574c75-329d-44a8-beed-fffe5e431e6e", //booking-transactionV4$.journeyInfo.offers[*].offerStoreId
                    "providerId": "100088", //booking-transactionV4$.journeyInfo.offers[*].provider (not provider ID, but provider slug eg. assertis)
                    "priceCents": 14082, //booking-transactionV4$.totalPriceInDisplayCurrency.lowestUnitValue
                    "feeCents": 0, //booking-transactionsV4$.priceBreakdown.{type==SERVICE_FEE}.price.lowestUnitValue
                    "fareTypeDisplayName": "First Class, Advance Single", //ticket-reservations$.reservations[*].fares.id
                    "outboundFareDetails": {
                        "class": "First Class", //ticket-reservations$.reservations[*].classes.name
                        "fare": "Advance Single", //ticket-reservations$.reservations[*].fares.name
                        "classRank": 0, //missing
                        "validity": [
                            //ticket-reservations$.reservations[*].fares.validityPolicy
                            "Valid only on the date and time shown on the ticket."
                        ],
                        "cancellation": [
                            //ticket-reservations$.reservations[*].fares.cancellationPolicy
                            "Ticket is non-refundable. Ticket can be exchanged for another Advance ticket, for the same journey and by the same route, for a fee and on payment of any difference in price; if the new ticket is cheaper then the difference won’t be refunded. The exchange must be made before the time on the original ticket. This can be done by making a replacement booking via your original point of sale and then contacting the Customer Service team."
                        ],
                        "amenities": [
                            //ticket-reservations$.reservations[*].classes.amenities
                            {
                                "id": "wifi.free",
                                "displayName": "Free Wi-Fi"
                            },
                            {
                                "id": "facility.powerPlug",
                                "displayName": "Power plugs"
                            },
                            {
                                "id": "facility.drinks",
                                "displayName": "Free drinks"
                            },
                            {
                                "id": "seat.large",
                                "displayName": "Wide seat"
                            },
                            {
                                "id": "facility.food",
                                "displayName": "Food available"
                            },
                            {
                                "id": "legroom.extra",
                                "displayName": "Extra legroom"
                            }
                        ],
                        "extraPriceInfo": [], //missing
                        "dynamicFareTerms": null, //ticket-reservations$.reservations[*].offerSpecs.dynamicTerm
                        "splitFareDetails": null //FIXME(nv): check if everything is covered
                    },
                    "policies": { //cannot find we map from search API
                        "cancellationPolicies": [
                            {
                                "cancellable": "NO",
                                "hoursBefore": 0,
                                "chargeFixedCents": 0,
                                "chargePercent": 0
                            }
                        ],
                        "modificationPolicies": [
                            {
                                "modifiable": "YES",
                                "hoursBefore": 0,
                                "chargeFixedCents": 1000,
                                "chargePercent": 0
                            }
                        ],
                        "cancellable": "NON_CANCELLABLE",
                        "modifiable": "SEMI_MODIFIABLE"
                    },
                    //FIXME(nv): link is created by us (should have all the data probably
                    "link": "https://www.omio.com.qa.goeuro.ninja/links/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXJ0bmVyIjp7ImlkIjoidWJlciJ9LCJlbmFibGVkIjp0cnVlLCJsYW5kVG8iOiJTRUFSQ0hfSk9VUk5FWV9QQUdFIiwic2tpcFRyYW5zZmVyUGFnZSI6dHJ1ZSwibGFuZGluZ0RvbWFpbiI6IiIsInBhcnRuZXJMb2dvVXJsIjoiIiwicHJpY2VDYXAiOjE0MDgyLCJwcm92aWRlcklkIjoiMTAwMDg4IiwiZGVwYXJ0dXJlRGF0ZSI6IjIwMjQtMDEtMDUiLCJlYXJsaWVzdERlcGFydHVyZVRpbWUiOiIwNzoxMyIsImFycml2YWxQb3NpdGlvbiI6MzgwNTc1LCJkZXBhcnR1cmVQb3NpdGlvbiI6MzgwNTUzLCJwYXNzZW5nZXJBZ2VzIjpbMzBdLCJyZXVzZVNlYXJjaCI6eyJzZWFyY2hJZCI6IlE1QTcyRDg0QUY1OTE0Q0M2QTEyOUI3QzZBNzQ3NDA5NCIsImpvdXJuZXlJZCI6IjEyNTA0NDczOTIiLCJvdXRib3VuZElkIjoiLTU5MjE4MjQxOTMyMDY3NDA5NTUiLCJpbmJvdW5kSWQiOiIifSwic291cmNlU3lzdGVtIjoiYjJiLWFwaS12MiIsImxvY2FsZSI6ImVuIiwiY3VycmVuY3kiOiJFVVIiLCJ0cmF2ZWxNb2RlIjoidHJhaW4ifQ.lhYO2BDhr6ZmwtVgnAD-9FsXB8aZBSvlgdKSoLYkVpc",
                    //possible to compute
                    "mobileTicketSupported": true,
                    //possible to compute
                    "segmentIds": [
                        1154224989
                    ],
                    //highly unlikely needed after booking
                    "availability": {
                        "displayMode": "DISPLAY_EXACT_NUMBER",
                        "totalTicketsLeft": 8
                    },
                    //available in booking
                    "permittedArrivalPositionIds": [],
                    //available in booking
                    "permittedDeparturePositionIds": []
                }
            ],
            //computable
            "journeyType": "oneway",
            //computable
            "cheapestPriceCents": 0
        }
    },
    //everything should be computable from tickets-reservationv2
    "ticketDetails": {
        //available
        "bookingReference": "1604",
        "confirmationEmail": "narek.vardanjan+doggio@omio.com",
        "ticketUrl": "/bookings/f1b40b15-46a6-4d78-b36a-b2e464c319c2/tickets",
        "mobileTicketUrl": "/bookings/f1b40b15-46a6-4d78-b36a-b2e464c319c2/mobile-tickets",
        "tickets": [
            {
                "id": "2d48e2cd-612b-411c-9c82-3c380e45e810",
                "status": "VALID",
                "referenceId": "H1EV74GN4CN",
                "ticketSegments": [
                    {
                        "segmentId": "1154224989",
                        "ticketUrls": [
                            "/bookings/f1b40b15-46a6-4d78-b36a-b2e464c319c2/mobile-tickets/1edef5cd-3289-4eb4-be11-e086bfc2525b",
                            "/bookings/f1b40b15-46a6-4d78-b36a-b2e464c319c2/pkpass/8002829f-6a2f-4ede-8e9d-da194f4c5780"
                        ]
                    }
                ]
            }
        ],
        "pkpassURLs": [
            "/bookings/f1b40b15-46a6-4d78-b36a-b2e464c319c2/pkpass/8002829f-6a2f-4ede-8e9d-da194f4c5780"
        ],
        "refundedAmountCents": null,
        "ticketTypes": null
    },
    //seems like only b2b thing
    "isSavedForLater": false,
    //couldn't find
    "sourceSystem": "b2b-booking-tool",
    //missing
    "journeyId": "g6NqaWTOSohQIKNvaWS0LTU5MjE4MjQxOTMyMDY3NDA5NTWjaWlkoA",
    //missing
    "bookingCodeHistory": null,
    "postSalesTicketURLs": null,
    "postSalesPriceAmends": []
}
```



# What we want to achieve

* Mostly backward compatible response format
* Life without corrupted bookings
* Speed that can compete with currently set-up SLAs
* On ticket modification merging positions, carriers, providers somewhere

# Different solutions

## Removing booking fragment and calling internal teams only (no cache)

* Call these internal service APIs
	* Tickets-reservation (50-60ms)
	* Booking-api (50-60ms)
	* Positions-api (180-220ms) for batch positions
	* Carriers / Providers CMS (can be probably cached locally from the directus CMS)
* Always do all the computations needed to serve the response

### Benefits

* Easy to implement, and test (just run everything locally no consumer data race with QA)
* No need to think about concurrency (we are not storing anything)

### Drawbacks
* Slower, depending on the position API can endanger SLAs
	* This can be resolved by asking booking to add positions also
* Not much wiggle room and slower development (adding a field to the response means always at least 2 tasks from the internal team, booking+search)
	* With fragments in case of quick fixing or accreditations, we could just wait for the search to complete and implement the rest on our side
* Ticket modification adds additional segments and positions to the response currently

## Removing booking fragment and calling internal teams only (with cache)

Same as the last one, but we would cache the response and only recompute based on updatedAt or state change from the booking response.

### Benefits
* Faster
### Drawbacks
* We need to get the caching straight 😄
* + Others from the previous solution

## The booking fragment stays, but consumer write-only and API mostly read-only

Partially implement the state machine from the booking-api.

booking-create (same, event not sent by booking)
* Create booking
* Poll from booking-api to initialized state
* Store in Couchbase the initial state
* Return result

booking-update (success means INITIALIZED or RESERVED)
* Update passenger details
* Poll from booking-api for INITIALIZED or RESERVED state
* Return result

booking-confirm
* Payment, Booking-Confirm
* Return BOOKING

booking-get
* Always from couchbase on our side
* No additional calls to internal teams (or maybe just booking-get)

### Benefits
* Fast get booking (most of the calls from UBER in the booking namespace)
![Pasted image 20240112135127.png](None)
* More in control of the data presented

### Drawbacks
* Harder to implement correctly
* We need to think well about the initial design (adding fields etc. should not affect these afterward)

