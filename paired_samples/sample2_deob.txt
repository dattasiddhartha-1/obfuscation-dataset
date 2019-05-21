# deob

#include <stdio.h>
int add(int);
int x, y, result;
int main(int argc, char **argv)
{
    if (argc != 3)
        return 0;
    argv++;
    x = atoi(*argv++);
    y = atoi(*argv++);
 add(add(add(add(add(add(add(add(add(add(add(add(add(add(add(add(0))))))))))))))));
    printf("%d\n", result);
    return 0;
}
int add(int carry)
{
    static int pos = -1;
    pos++;
    int b1 = !!(x & 1 << pos);
    int b2 = !!(y & 1 << pos);
    result |= ((b1 ^ b2) ^ carry) << pos;
    return ((b1 && b2) || (carry && (b1 ^ b2)));
}