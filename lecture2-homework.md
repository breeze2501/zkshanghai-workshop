# 第2课 课后作业

## 转换为bit位 Num2Bits

```circom
pragma circom 2.1.4;

template Num2Bits (nBits) {
    signal input num;
    signal output b[nBits];

    var acc = 0;
    for (var i = 0; i < nBits; i++) {
        b[i] <-- (num \ (2 ** i)) % 2;
        (1 - b[i]) * b[i] === 0;
        acc += (2 ** i) * b[i];
    }
    acc === num;
}

component main = Num2Bits(5);

/* INPUT = {
    "num": "1",
    "b": ["1", "0", "0", "0", "0"]
} */

```

## 判零 IsZero

```circom
pragma circom 2.1.4;

template IsZero () {
    signal input in;
    signal output out;

    signal inv;
    inv <-- in == 0 ? 0: 1/ in;

    out  <== - in * inv + 1;
    in * out === 0;
}

component main = IsZero();

/* INPUT = {
    "in": "4",
    "out": "0"
} */

```

## 相等 IsEqual

```circom
pragma circom 2.1.4;

template IsZero () {
    signal input in;
    signal output out;

    signal inv;
    inv <-- in == 0 ? 0: 1/ in;

    out  <== - in * inv + 1;
    in * out === 0;
}

template IsEqual () {
    signal input in[2];
    signal output equal;

    component isZero = IsZero();
    isZero.in <== -in[0] + in[1];
    equal <== isZero.out;
}

component main = IsEqual ();

/* INPUT = {
    "in": ["4", "4"],
    "out": "1"
} */
```

## 选择 Selector

```circom
pragma circom 2.1.4;

template IsZero () {
    signal input in;
    signal output out;

    signal inv;
    inv <-- in == 0 ? 0: 1/ in;

    out  <== - in * inv + 1;
    in * out === 0;
}

template IsEqual () {
    signal input in[2];
    signal output equal;

    component isZero = IsZero();
    isZero.in <== -in[0] + in[1];
    equal <== isZero.out;
}

template CalculateTotal(n) {
    signal input in[n];
    signal output out;

    signal sums[n];

    sums[0] <== in[0];

    for (var i = 1; i < n; i++) {
        sums[i] <== sums[i-1] + in[i];
    }

    out <== sums[n-1];
}

template Selector(choices) {
    signal input in[choices];
    signal input index;
    signal output out;

    component eqs[choices];
    component cal = CalculateTotal(choices);

    // For each item, check whether its index equals the input index.
    for (var i = 0; i < choices; i ++) {
        eqs[i] = IsEqual();
        eqs[i].in[0] <== i;
        eqs[i].in[1] <== index;

        cal.in[i] <== eqs[i].equal * in[i];
    }

    // Returns 0 + 0 + 0 + item
    out <== cal.out;
}

component main = Selector(2);

/* INPUT = {
    "in": ["3", "4"],
    "index": "1",
    "total": "4",
    "out": "4"
} */

```
