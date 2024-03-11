# Black Jack

```
points: 447
text:
As I stepped into the bustling casino, the air was thick with anticipation. My eyes scanned the room until they landed on the blackjack table. Sitting across from me was the dealer, their movements precise and mechanical. With each shuffle and deal, there was something eerily robotic about them, sending a chill down my spine. Yet, I couldn't resist the allure of the cards spread out before me, beckoning me to take a chance.

nc 35.94.129.106 3002
```

## TL;TR
looking at the disassebled binary we can see that it uses an lcg for the rng and the **current time to the second as the seed**.
that can be used to predict every card that will show up in the game by reversing the rng and B or maybe we could open a session and see if going back with the timestampo we hit the right seed Blah Blah Blah.

### there are people who have reversed this challenge and wrote a script  

### there are people who failed this chall

## AND THEN, THERE IS ME i got bored and opened 2 connections at the same time. 

![Alt text](./blackjack.png) 

on one connection we just always bet 0 to see the cards (on the right)

- if we **win** on the fake connection then on the real one we **bet all the budget**

- if we **lose** or **push** on the fake connection then we replay those same moves but **betting 0**

*then spend some time practicing your blackjack skills*

i find this much more convenient as reversing the rng would still require writing a script or playing by hand 

### ✨✨*make it simple not harder*✨ ✨
![Alt text](./image-5.png)