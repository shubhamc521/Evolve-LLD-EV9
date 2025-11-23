# KISS Principle (keep it Simple, Stupid)

## One Liner
Favors simple, clear solution over complicated onces.
Simple code is easier to read, test and maintain.


## Bad Example

```java
public int sum(int[] arr) {
    return Arrays.stream(arr).sum();
}
```

This works, but everyone won't be able to understand it directly.
We need extra efforts to read and maintain.

## Good Example
```java
public int sum(int[] arr) {
    int total = 0;
    for(int num : arr){
        total += num;
    }
}
```