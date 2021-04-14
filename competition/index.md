WORK IN PROGRESS, This is just a draft.

Disclaimer: The information on this page is not normative. It might be outdated (if you notice this, please fix it or create an issue!). In any case, official documents (such as a Call for Pariticipation, the Rule Book, or on-site announcements) take precedence.



## Qualification Process

At some point in time (TODO: more concrete), a Call for Participation/Call for Applications is sent out. It will contain all necessary information.

...

## Registration

Qualified teams usually get a registration code. Details may differ each year, but you will get this information anyway.

## TDPs/TRRs/Posters

- TDP = Team Description Paper
- TRR = Team Research Report
- There is no such thing as a TDP in the SPL (anymore)
  - TODO: explain what this was and why it's not done anymore
  - The poster (see rule book section A.1) replaced this in 2018/2019
- TRRs are required for pre-qualified teams, but are related to the year in which the pre-qualification was gained (and not to the year for which the qualification is done)
- Of course, you can publish a TDP any time you like. It just isn't included in any official proceedings.


## Team Number

Every SPL team that participates in an official RoboCup competition is assigned a "team number". This number [...]. It [...]. However, since the team number should be in the range [1,89], we reserve the right to reassign a team number if the team that had this number assigned did not participate in any RoboCup competition for several years.

The true list of teams and team numbers is [here in the GameController repository](https://github.com/RoboCup-SPL/GameController/blob/master/resources/config/spl/teams.cfg). [See also](https://github.com/RoboCup-SPL/GameController#adding-teams-to-the-gamecontroller).

The team number is used in multiple places:
- GameController messages
- team communication: teamNum field, port (10000 + team number)
- network setup: usually third component of IP: `10.0.<teamNumber>.<robot>` (not specified in the rules, but is usually done like this)

## GameController

The GameController is the application that is required by the rule book to manage the game and communicate referee decisions to the players.

- on [GitHub](https://github.com/RoboCup-SPL/GameController)
- special releases for RoboCup competitions are made and announced on mailing lists (which?)
- interface to robots: `include/RoboCupGameControlData.h`
  - GameController broadcasts `RoboCupGameControlData` on port `GAMECONTROLLER_DATA_PORT` (UDP)
  - players should reply with `RoboCupGameControlReturnData` on port `GAMECONTROLLER_RETURN_PORT` (UDP) *unicast to the origin of the GameController packets* to show up in the GameController

## Network Setup

TODO: generic stuff about WiFi, IPs, when to be in a network, don't use another team's team number, don't use other WiFi/2.4GHz/5GHz equipment etc. etc.

A common setup is that each field has its own WiFi access point with its own SSID. Often, the field networks are separated, so robots on different fields can't communicate with each other. However, it is also possible that at a particular venue all fields are in the same network. Therefore, you have to make sure that your robots only listen to GameController messages for your team (i.e. your team number matches one of those in `RoboCupGameCtrlData.teams[0|1].teamNumber`.

TODO: 2021 special stuff

## Streaming

TODO
