# elevator problem

<details >
    <summary>Elevator Problem statement</summary>

You are tasked with the job designing and writing the decision making logic for an elevator in a 5-floor building. The software receives inputs from the following sources:

1. A panel of five push buttons inside the elevator car to select a destination floor.
1. Two push buttons (up/down) on each floor to request the elevator. (Remark: the top and bottom floors obviously only have one button).
1. A sensor on each floor to indicate that the elevator has arrived at that floor (triggered when the elevator is in motion).
1. A "timer expired" event that occurs to indicate the passage of time. This is used to hold the doors open for a set amount of time during loading for instance.

The elevator operates according to the "Elevator Algorithm" as described at https://en.wikipedia.org/wiki/Elevator_algorithm. In a nutshell, elevators work by alternating between upward and downward motion. When moving up, the elevator keeps moving up until there are no more floors to service. It then reverses direction and moves down until there are no more floors to service. A key aspect of this approach is that it avoids starvation. For example, suppose the elevator is on floor 1 and Bob presses "down" on floor 3. The elevator will start moving up, but suppose that Alice now presses "down" on floor 5. The elevator will _pass_ floor 3 and go all the way up to floor 5 first. It will then stop to pick up Bob on the return trip.

The elevator has a number of features generally related to "quality of service" and safety.

1. The elevator should not cut people in half by suddenly moving with the door open.
1. The elevator should not shoot through the ceiling of the building or burrow into the ground.
1. Someone who has requested the elevator should be guaranteed to get the elevator.

### YOUR TASK:

Design and implement code for the decision making logic of the elevator. More importantly, come up with a strategy for testing it.

### A CHALLENGE:

To write this code you might ask to know more about how the elevator hardware control actually works. For example, an elevator obviously has a motor that moves the car up and down. There is a door that opens and closes. There is a timer that can be set. Are these devices that receive commands? If so, how does that work? Are we somehow responsible? Unfortunately, we just don't have any information about that aspect of the elevator--that's a different corporate division.

Thus, one tricky part of the project is to think about how what aspects of the elevator system are truly essential to the problem at hand. We'll also need to consider the way in which the logic will be used by other software (not shown/provided). It also means that our understanding of the problem might be incomplete and that we should try to write the code in a way that allows it to be extended to handle new "requirements."

### HINTS:

At first, this problem is likely to seem overwhelming. There are many moving parts. Where to even begin?

When presented with a problem like this, it might help to slightly "underthink" the problem. What is the fundamental problem being solved? What is the least amount of information you need to solve that problem? What is the simplest thing that you can actually code? What can you actually test?

With that in mind, here are some specific things to focus on:

1. Operation. What does an elevator actually do when it operates?
1. State. What information minimally needs to be stored?
1. Inputs. What inputs does the elevator software receive?
1. Outputs. What outputs are going to be produced?
1. Invariants. What is never supposed to happen?

Use your intuition and your experience as a user of an elevator. Also, try to build your initial understanding of the problem at a high level without immediately jumping into code. Code can come later.

</details>

## Requirements

An elevator needs to be designed for a 5-floor building with the basic functionality of being able to be requested from all floors. The elevator should also implement safety features to avoid cases like going through the roof or burrowing itself underground.
The implementation should be kept open-ended as more features are likely to be requested. The integration details of this elevator with the additional components like the door are unknown as of now, so that should be accounted for while designing it.

## Approach

### Components and their responsibilities

1. Panel inside the elevator
   - Maintain the following state
     - current-floor
     - active-queue (stores requests currently being served by the elevator)
     - backlog-queue (stores requests that will be attended to once the active queue is empty)
   - Respond to the following events
     - user request from any floor
     - when elevator reaches any floor
     - floor button press inside the elevator
1. User controls at each floor
   - store its own floor number
   - when lift is requested, send its own floor number and request direction to the panel
1. Sensors at each floor
   - sends elevator reached event whenever the elevator crosses a floor

### Simplest version of the problem: Taking the user to the requested floor

User on floor 5 intends to go to floor 0

#### Solution:

- The elevator is stationary at floor 2
- Users requests to go down from floor 5
- Elevator responds to the request event by noting the request floor
- Elevator moves from floor 2 to floor 5, picks up the user
- The user requests to go floor 0 from inside the lift
- The lift moves to floor 0

### Feature: Handling requests from multiple users

user_a on floor 5 intends to go down. While user_a is in the elevator, user_b requests to go down from floor 3.

- Elevator receives request of user_a from floor 5, stores the request floor and direction in a request queue and moves to floor 5.
- user_a enters the elevator and presses floor 0
- While the elevator is crossing floor 4, user_b requests from floor 3 to go down. Since the current request and latest request are in the same direction, the elevator stops at floor 3 to pick up user_b
- The elevator drops both the users at floor 0 and waits for the next request
- Note: if user_b (latest request) would've wanted to go upwards (direction opposite to the current user), the elevator would have stored this request as its next task. This is done to prioritize the request of the current user of the elevator

## Pseudocode

### State maintained by the elevator panel

1. current-floor
1. active-queue
1. backlog-queue

### External event: User request from any floor

_it is a matter of selecting whether to store the current request in active-queue or backlog-queue_

1. check if active-queue is empty
   - if yes, then push the request-floor and request-direction to active-queue
   - else if active-queue[0].direction == request-direction
     - check if (request.direction == "up" and current-floor <= request-floor)
       - push request-floor and request-direction to active-queue and sort active-queue in ascending order
     - else if (request.direction == "down" and current-floor > request-floor)
       - push request-floor and request-direction to active-queue and sort active-queue in descending order
   - else push the request-floor and request-direction to backlog-queue
1. Update the active-queue and backup-queue in backup state
1. move the elevator to active-queue[0].floor

### External event: User request from inside the elevator

1. check if current-floor > request-floor
   - if yes, direction = "down"
   - else direction = "up"
1. trigger user request from a floor event using the following parameters:
   - request-floor (the number pressed by the user)
   - direction (calculated in the above step)

### Internal event: Move elevator to floor n

_a recursive call to move the elevator up or down depending on current-floor_

1. check if current-floor > n
   - if yes, move elevator down (takes 3 sec)
   - else if current-floor < n, move elevator up (takes 3 sec)
   - else (lift is at the desired floor )
     - remove the first request from active-queue
     - open the doors and wait for 5 sec
     - check if active-queue is empty
       - if yes, then move the requests having same direction from backlog queue to active queue
     - move the elevator to active-queue[0].floor

### External event: lift reached a floor n

1. Update the current-floor to n
1. Update the current-floor in backup state
