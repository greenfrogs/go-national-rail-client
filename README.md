# go-national-rail-client

[![Go Report Card](https://goreportcard.com/badge/github.com/martinsirbe/go-national-rail-client)](https://goreportcard.com/report/github.com/martinsirbe/go-national-rail-client)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fmartinsirbe%2Fgo-national-rail-client.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fmartinsirbe%2Fgo-national-rail-client?ref=badge_shield)
[![codecov](https://codecov.io/gh/martinsirbe/go-national-rail-client/branch/main/graph/badge.svg)](https://codecov.io/gh/martinsirbe/go-national-rail-client)

An open-source Golang library for interfacing with the National Rail's Darwin OpenLDBWS web service. This client 
enables the retrieval of real-time train information, focusing on departures and arrivals between specific stations.

The client have been developed based on [Live Departure Boards Web Service][1] documentation.

## Getting Started
To use this client, register for an OpenLDBWS access token at [National Rail Enquiries][2].

## Configuration

The API client can be configured using the following options:
- **AccessTokenOpt**: Set using `AccessTokenOpt("YOUR_ACCESS_TOKEN")`. If not provided, the library attempts to 
retrieve the access token from the `NR_ACCESS_TOKEN` environment variable. This token is crucial for API authentication.  
Client initialisation will fail if neither `AccessTokenOpt` nor `NR_ACCESS_TOKEN` environment variable is provided.

- **HTTPClientOpt**: Customised using `HTTPClientOpt(customHttpClient)`. If not provided, a default HTTP client with 
the following configuration is used:
    - `Timeout`: 10 seconds
    - `MaxIdleConns`: 10 connections
    - `MaxConnsPerHost`: 10 connections
    - `MaxIdleConnsPerHost`: 10 connections

- **URLOpt**: Set using `URLOpt("https://example.com")`. If not provided, will use `https://lite.realtime.nationalrail.co.uk/OpenLDBWS/ldb11.asmx`.

## Examples
### Obtain Departure Board
To retrieve the departure board for a specific train station, provide the station code and the desired number of 
departures. This example demonstrates how to obtain the departure board for Gillingham (Kent) [GLM], displaying 5 
upcoming train departures.

```golang
package main

import (
  "fmt"

  nr "github.com/martinsirbe/go-national-rail-client/nationalrail"
)

func main() {
  client, err := nr.NewClient(
    nr.AccessTokenOpt("e995265f-df60-4787-bafc-af5a433f9b22"),
  )

  board, err := client.GetDepartures(nr.StationCodeGillinghamKent, nr.NumRowsOpt(5))
  if err != nil {
    panic(err)
  }

  fmt.Printf("%s [%s] Departure Board:\n", board.LocationName, board.CRS)
  fmt.Println("----------------------------------------")
  fmt.Println("Time  | Platform | Status  | Destination")
  fmt.Println("----------------------------------------")
  for _, s := range board.TrainServices {
    platform := "?"
    if s.Platform != nil {
      platform = *s.Platform
    }

    fmt.Printf("%s |    %s     | %s | %s [%s]\n", s.STD, platform, s.ETD, s.Destination.Name, s.Destination.CRS)
  }
}
```

Output:
```shell
Gillingham (Kent) [GLM] Departure Board:
----------------------------------------
Time  | Platform | Status  | Destination
----------------------------------------
12:57 |    2     | On time | London Cannon Street [CST]
13:04 |    2     | On time | Kentish Town [KTN]
13:07 |    3     | On time | Ramsgate [RAM]
13:10 |    2     | On time | London Cannon Street [CST]
13:11 |    3     | On time | Rainham (Kent) [RAI]
```

### Obtain Fastest Departures
```golang
package main

import (
	"fmt"

	nr "github.com/martinsirbe/go-national-rail-client/nationalrail"
)

func main() {
	client, err := nr.NewClient(
		nr.AccessTokenOpt("e995265f-df60-4787-bafc-af5a433f9b22"),
	)

	board, err := client.GetFastestDepartures(
		nr.StationCodeGillinghamKent,
		nr.FilterDestinationsOpt([]nr.CRSCode{
			nr.StationCodeRochester,
			nr.StationCodeSittingbourne,
		}),
	)
	if err != nil {
		panic(err)
	}

	fmt.Printf("%s [%s] Departure Board:\n", board.LocationName, board.CRS)
	fmt.Println("----------------------------------------")
	fmt.Println("Time  | Platform | Status  | Destination")
	fmt.Println("----------------------------------------")
	for _, s := range board.TrainServices {
		platform := "?"
		if s.Platform != nil {
			platform = *s.Platform
		}

		fmt.Printf("%s |    %s     | %s | %s [%s]\n", s.STD, platform, s.ETD, s.Destination.Name, s.Destination.CRS)
	}
}
```

Output:
```shell
Gillingham (Kent) [GLM] Departure Board:
----------------------------------------
Time  | Platform | Status  | Destination
----------------------------------------
20:04 |    2     | On time | Kentish Town [KTN]
20:06 |    3     | On time | Ramsgate [RAM]
```

## License
See the [LICENSE](LICENSE.md) file for license rights and limitations (MIT).

[1]: https://lite.realtime.nationalrail.co.uk/openldbws/
[2]: https://realtime.nationalrail.co.uk/OpenLDBWSRegistration/
