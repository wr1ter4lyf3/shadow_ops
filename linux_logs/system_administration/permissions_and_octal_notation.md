## Octal Notation

Three-digit number<br>

Each digit reps perms as sum of 4 addends corresp. to **read (r,4), write (w,2), & execute (x,1)** perms. <br>

The first digit applies to **user (owner) (u)** <br>
The second digit --> **group (g)** <br>
Third digit --> the world /**other users (o)** <br>

| Octal Digit | Permission(s) Granted | Symoblic |
|---|---|---|
| 0 | None | `[u/g/o/]`-rwx
| 1 | Execute permission only | `[u/g/o]`=x
| 2 | Write permission only | `[u/g/o]`=w
| 3 | Write and execute permission only: 2+1=3| `[u/g/o]`=wx
| 4 | Read permission only| `[u/g/o]`=r
| 5 | Read and execute permissions only 4+1=5| `[u/g/o] `=rx
| 6 | Read and write permissions only;4+2=6| `[u/g/o]`=rw
| 7 | All permission (read, write, execute); 4+2+1=7| `[u/g/o]`=rwx

<img width="808" height="555" alt="image" src="https://github.com/user-attachments/assets/0deef097-ff7d-41f1-9c66-c2d90e33d38c" />
