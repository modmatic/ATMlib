### FILE/ARRAY FORMAT DESCRIPTION

| **Section**              | **Field**             | **Type**        | **Description** |
| ---                      | ---                   | ---             | ---             |
| **Track table**          |                       |                 | **Number of tracks and their addresses** |
|                          | Track count           | UBYTE (8-bits)  | Number of tracks in the file/array |
|                          | Address track 1       | UWORD (16-bits) | Location in the file/array for track 1 |
|                          | …                     | …               | … |
|                          | Address track *__N__* | UWORD (16-bits) | Location in the file/array for track *__N__ (0 … 255)* |
|   |
| **Channel entry tracks** |                       |                 | **For each channel, track to start with** |
|                          | Channel 0 track       | UBYTE (8-bits)  | Starting track index for channel 0 |
|                          | …                     | …               | … |
|                          | Channel 3 track       | UBYTE (8-bits)  | Starting track index for channel 3 |
|   |
| **Track 0**              |                       |                 | **Commands and parameters for track 0** |
|                          | Command 0             | UBYTE (8-bits)  | See command list |
|                          | *and its* Parameters  | none/variable   | *See __parameter list__ for each command* |
|                          | …                     | …               | … |
|                          | Command N             | UBYTE (8-bits)  | |
|                          | *and its* Parameters  | none/variable   | |
| **…**                    | **…**                 | **…**           | **…** |
| **Track _N_**            |                       |                 | **Commands and parameters for track _N_** *(0-255)* |


### COMMAND LIST

| **Command (_X_)** | **Parameter**        | **Type**           | **Description** |
| ---               | ---                  | ---                | ---             |
|                0  |                      |                    | Stop playing    |
|   |
|           1 …  63 |                      |                    | Start playing note *[__X__]* where 1 is a C1. See [Frequency to Tone](./frequencyToTone.md "Frequency to Tone table")|
|   |
|          64 … 159 |                      |                    | Configure effects (fx) |
|                   | *See __fx list__*    | none/variable      | Effect is *[__X__ - 64]* |
|   |
|         160 … 223 |                      |                    | Delay for *[__X__ - 159]* ticks |
|   |
|               224 |                      |                    | Long delay |
|                   | Ticks (*__Y__*)      | VLE (8/16/24-bits) | Delay for *[__Y__ + 64]* ticks |
|   |
|     ~~225 … 251~~ |                      |                    | ~~RESERVED~~ |
|   |
|               252 |                      |                    | Call/run/goto specified track |
|                   | Track                | UBYTE (8-bits)     | Track index |
|   |
|               253 |                      |                    | Repeated call/run/goto specified track |
|                   | Loop count (*__Y__*) | UBYTE (8-bits)     | Repeat *[__Y__ + 2]* times (total) |
|                   | Track                | UBYTE (8-bits)     | Track index |
|   |
|               254 |                      |                    | Return/end of track marker |
|   |
|               255 |                      |                    | Binary data |
|                   | Length               | VLE (8/16/24-bits) | Length in bytes of data to follow |
|                   | Data                 | variable           | Binary data chunk (notify host application) |


### FX LIST

| **Effect** | **Parameter**    | **Type**      | **Description** |
| ---        | ---              | ---           | ---             |
| **64+0**      | set Volume (*__Y__*) | UBYTE (8-bit) | Set volume to *[__Y__]*. <br /> **_Note:_**<br /> If the combined volume of all channels exceed 255 there may be rollover distortion. This should not be disallowed, as it may be usesful as an effects hack for the musician. There should however be a non-interfering warning when a musician enters a value above 63 for ch 1-3 or 32 for ch 4 (noise). ch 4 the volume is counted double, so 32 is actually 64 |
| **64+1**      | slide Volume ON (*__Y__*) | UBYTE (8-bit) | Slide the volume with an amount (positive or negative) of *[__Y__]* for every tick.<br /> **_Note:_**<br />  This results in a fade-in or fade-out effect. The volume is not limited, but rols over when it exceeds 127 or goes below 0. However there should be a non-interfering warning when sliding would result in exceeding 63 for ch 1-3 and 32 for ch 4. |
| **64+2**      | slide Volume ON advanced (*__Y__*) (*__Z__*)| UBYTE (8-bit) UBYTE (8-bit) |  Slide the volume with an amount (positive or negative) of *[__Y__]* for every [*__Z__*] ticks.<br /> **_Note:_**<br />  This results in a fade-in or fade-out effect. The volume is not limited, but rols over when it exceeds 127 or goes below 0. However there should be a non-interfering warning when sliding would result in exceeding 63 for ch 1-3 and 32 for ch 4. |
| **64+3**      | slide Volume OFF |  |  stops the volume slide |
| **64+4**      | slide Frequency ON (*__Y__*) | UBYTE (8-bit) | Slide the frequency with an amount (positive or negative) of *[__Y__]* for every tick.<br /> **_Note:_**<br /> The amount of slide is limited between -127 to 127|
| **64+5**      | slide Frequency ON advanced (*__Y__*) (*__Z__*)| UBYTE (8-bit) UBYTE (8-bit) |  Slide the frequency with an amount (positive or negative) of *[__Y__]* for every [*__Z__*] ticks.<br /> **_Note:_**<br /> The amount of slide and ticks devider is limited between -127 to 127 |
| **64+6**      | slide Frequency OFF |  |  stops the frequency slide |
| **64+7**      | set Arpeggio (*__X__*)(*__Y__*) | UBYTE (8-bit) UBYTE (8-bit) | Next to the current playing note, play a second and third note *[__X__]* for every *[__Y__]* ticks. *[__X__]* includes 2 parameters: AAAABBBB, where AAAA = base + amount to second note and BBBB = second note + amount to third note.<br /> *[__Y__]* includes 4 parameters: FEDCCCCC, where F = reserved, E = toggle no third note, D = toggle retrigger, CCCCC = tick amount.<br />**_Note:_**<br />Arpeggio is used for playing 3 notes out of a chord indivually |
| **64+8**      | Arpeggio OFF |  | stops the arpeggio |
| **64+9**      | set Retriggering (*__X__*) | UBYTE (8-bit) | Noise channel consists of white noise. By setting retriggering *[__X__]* it swithes the entrypoint at a given speed. *[__X__]*  includes 2 parameters: AAAAAABB , where AAAAAA = entry point and BB = speed (0 = fastest, 1 = faster , 2 = fast)|
| **64+10**      | Retriggering off | | stops the retriggering for the noise on channel 3 |
| **64+11**      | add Transposition (*__X__*)| UBYTE (8-bit) | shifts the played notes by adding *[__X__]* to the existing transposition for all playing notes.<br />**_Note:_**<br /> The amount of shift is limited between -127 to 127. However there should be a non-interfering warning when transposing would result in exceeding 63 or get lower than 0 |
| **64+12**      | set Transposition (*__X__*)| UBYTE (8-bit) | shifts the played notes by setting the transposition to [__X__]* for all playing notes.<br />**_Note:_**<br /> The amount of shift is limited between -127 to 127. However there should be a non-interfering warning when transposing would result in exceeding 63 or get lower than 0 |
| **64+13**      | Transposition OFF |  | stops the transposition |
| **64+14**      | set Tremolo or Vibrato (*__Y__*)(*__Z__*) |UBYTE (8-bit) UBYTE (8-bit)|*[__Y__]* sets Depth.<br /> *[__Z__]* includes 4 parameters: RTxBBBBB Retrig, TremoloOrVibrato, reserved , rate<br />**_Note:_**<br /> Tremolo and Vibrato can **NOT** be combined|
| **64+15**      | Tremolo or Vibrato OFF|  | stops the tremolo or vibrato |
| **64+16**      | SET Glissando (*__X__*)| UBYTE (8-bit) | *[__X__]* includes 2 parameters: VTTTTTTT  Value ( 0 = go 1 note up, 1 = go 1 note down) and Ticks (amount of ticks, between each step) |
| **64+17**      | Glissando OFF|  | stops the Glissando |
| **64+18**      | SET Note Cut (*__X__*)(*__Y__*)| UBYTE (8-bit) UBYTE (8-bit) | *[__X__]* using 0xFF activates Note Cut and *[__Y__]* sets the equal amount of ticks between note ON and OFF
| **64+19**      | Note Cut Off|  | stops the Note Cut |
| ~~TBD~~    | ~~TBD~~          | ~~TBD~~       | ~~TBD~~         |



#### Thoughts on effects:

**Note:** These are the primitives to be implemented in the playroutine effects processor. Most will have several effect command numbers associated with them for various aspects of the same primitive. Effects can be combined but not stacked, but some combinations may have undesired/interesting interference.

* Volume slide: a gradual increasing or decreasing of the volume.
* Frequency slide: a gradual increasing or decreasing of the [frequency](https://en.wikipedia.org/wiki/Frequency "frequency wikipedia").
* Portamento: a gradual slide from one [note](https://en.wikipedia.org/wiki/Musical_note "note wikipedia") to another [note](https://en.wikipedia.org/wiki/Musical_note "note wikipedia").
* Arpeggio: a group of [notes](https://en.wikipedia.org/wiki/Musical_note "note wikipedia") which are rapidly and automatically played one after the other.
* Retriggering (on [note](https://en.wikipedia.org/wiki/Musical_note "note wikipedia") or by automation): oscillators are restarted either automatically or at the start of each new note.
* Transposition (also for microtonals): play [notes](https://en.wikipedia.org/wiki/Musical_note "note wikipedia") in a different key, or fine tune notes to provide microtonals; frequencies that are in between notes.
* Tremolo: a slight, rapid and regular fluctuation in the amplitude/volume of a [note](https://en.wikipedia.org/wiki/Musical_note "note wikipedia").
* Vibrato: a slight, rapid and regular fluctuation in the [pitch](https://en.wikipedia.org/wiki/Pitch_(music) "pitch wikipedia") of a [note](https://en.wikipedia.org/wiki/Musical_note "note wikipedia").
* Glissando: controls if and how a gradual frequency slide "snaps" to adjacent notes.
* Note cut (with delay and automation): provides a method to stutter and adjust note timing.
* Envelopes (instruments): the [attack, sustain, and decay](https://en.wikipedia.org/wiki/Synthesizer#Attack_Decay_Sustain_Release_.28ADSR.29_envelope "envelope wikipedia") of a sound.

