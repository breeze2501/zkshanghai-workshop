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

## 判负 IsNegative

```circom
pragma circom 2.1.4;

include "circomlib/poseidon.circom";
// include "https://github.com/0xPARC/circom-secp256k1/blob/master/circuits/bigint.circom";

template Num2Bits (nBits) {
    signal input in;
    signal output out[nBits];

    var acc = 0;
    for (var i = 0; i < nBits; i++) {
        out[i] <-- (in \ (2 ** i)) % 2;
        (1 - out[i]) * out[i] === 0;
        acc += (2 ** i) * out[i];
    }
    acc === in;
}

template CompConstant(ct) {
    signal input in[254];
    signal output out;

    signal parts[127];
    signal sout;

    var clsb;
    var cmsb;
    var slsb;
    var smsb;

    var sum=0;

    var b = (1 << 128) -1;
    var a = 1;
    var e = 1;
    var i;

    for (i=0;i<127; i++) {
        clsb = (ct >> (i*2)) & 1;
        cmsb = (ct >> (i*2+1)) & 1;
        slsb = in[i*2];
        smsb = in[i*2+1];

        if ((cmsb==0)&&(clsb==0)) {
            parts[i] <== -b*smsb*slsb + b*smsb + b*slsb;
        } else if ((cmsb==0)&&(clsb==1)) {
            parts[i] <== a*smsb*slsb - a*slsb + b*smsb - a*smsb + a;
        } else if ((cmsb==1)&&(clsb==0)) {
            parts[i] <== b*smsb*slsb - a*smsb + a;
        } else {
            parts[i] <== -a*smsb*slsb + a;
        }

        sum = sum + parts[i];

        b = b -e;
        a = a +e;
        e = e*2;
    }

    sout <== sum;

    component num2bits = Num2Bits(135);

    num2bits.in <== sout;

    out <== num2bits.out[127];
}

template IsNegative () {
    signal input in;
    signal output out;

    component n2b = Num2Bits(254);

    n2b.in <== in;

    component cmp = CompConstant(10944121435919637611123202872628637544274182200208017171849102093287904247808);
    
    for (var i = 0; i < 254; i++) {
        n2b.out[i] ==> cmp.in[i];
    }

    out <== cmp.out;
}

component main = IsNegative();

/* INPUT = {
    "in": "1",
    "out": "0"
} */

```

不能使用LessThan因为超出了LessThan输入信号的范围，不能使用比较器是因为（TODO）

## 少于 LessThan

```circom
template LessThan(n) {
    assert(n <= 252);
    signal input in[2];
    signal output out;

    component n2b = Num2Bits(n+1);

    n2b.in <== in[0]+ (1<<n) - in[1];

    out <== n2b.out[n] - 1;
}
```

## 整数除法 IntegerDivide
