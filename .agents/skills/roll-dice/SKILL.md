---
name: roll-dice
description: Roll dice using a random number generator. Use when asked to roll a dice, or generate a random dice roll.
---

To roll a die, use the following command that generates a random number from 1 to the given number of slides:

'''bash
echo $((RANDOM % <sides> + 1))
'''

'''powershell
Get-Random -Minimum 1 -Maximum (<sides> + 1)
'''

Replace '<sides>' with the number of sides on the die (e.g., 6 for standard die, 20 for a d20).