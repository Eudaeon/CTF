# Scrambled Payload

## Recon

VBS script.

```plaintext
┌──(quickemu㉿hacking-kali)-[/workspace]
└─$ file payload.vbs               
payload.vbs: ASCII text, with very long lines (60674), with no line terminators
```

## Analysis

Code is obfuscated with multiple layers of Base64 encoding and arithmetic operations.

In the end, it verifies that a string matches the regexes:

1. `^[MSy][FfK][ERT][yCM][efI][{31][KeN][jIS][Uol][z5j][}TR][DNV][4Qj][kY_][{Qw][Qz9][R{h][UF_][9Ns][l7W][SQI][lPb][9ZQ][QTJ][Y97][Ei3][IKL][x0U][iUX][FOE][QnU][xL8][RT_][lkL][d}q][9Sa]$`
2. `^[{Sp][F7H][R1t][CHG][ze5][1na][D7N][jGJ][U}r][kBj][RSq][ZEN][3WQ][k9q][Kw9][XzV][WkR][FLi][m94][HW2][dQT][r{l][9}t][tpT][B8Y][A13][TI0][M7x][EZU][yFb][Quh][BRx][TsA][kQJ][3Xd][r39]$`
3. `^[WoS][cFe][_yR][CzE][Xce][1HN][OYN][vTj][uDU][MYj][Rr7][GN4][tEQ][8kd][wnr][zpI][5Ra][F2x][9hP][xeW][9JQ][lRF][9ai][j7T][UVY][c3F][enI][fwx][vUH][xXF][Q1{][EVx][5TX][Fki][Zdw][of9]$`

The string should be `SFRCe1NjUjRNQkwzRF9WQl9TY3IxUFQxTkd9`.

Flag: `HTB{ScR4MBL3D_VB_Scr1PT1NG}`
