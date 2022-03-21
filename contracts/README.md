## Contract Indice

### Compile Indice contract 
- generates michelson code 
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile contract views_hangzhou/jsligo/indice.jsligo -e indiceMain --protocol hangzhou > views_hangzhou/jsligo/compiled/indice.tz
```
- generates michelson code in JSON format
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile contract views_hangzhou/jsligo/indice.jsligo --michelson-format json -e indiceMain --protocol hangzhou > views_hangzhou/jsligo/compiled/indice.json
```

### Compile Indice storage
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile storage views_hangzhou/jsligo/indice.jsligo '4' -e indiceMain --protocol hangzhou
```

### Compile parameter (with ligo compiler) into Michelson expression

- For entrypoint Increment
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile parameter views_hangzhou/jsligo/indice.jsligo 'Increment(5)' -e indiceMain --protocol hangzhou
```
- For entrypoint Decrement
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile parameter views_hangzhou/jsligo/indice.jsligo 'Decrement(5)' -e indiceMain --protocol hangzhou
```


### Simulate execution of entrypoints (with ligo compiler)

- For entrypoint Increment
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next run dry-run views_hangzhou/jsligo/indice.jsligo  'Increment(5)' '37' -e indiceMain --protocol hangzhou
```

- For entrypoint Decrement
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next run dry-run views_hangzhou/jsligo/indice.jsligo  'Decrement(5)' '37' -e indiceMain --protocol hangzhou
```

### Originate the Indice contract (with tezos-client CLI)
```
tezos-client originate contract indice transferring 1 from bootstrap1 running '/home/frank/Marigold/advisor/views_hangzhou/jsligo/compiled/indice.tz' --init '0' --dry-run
```


### Unit test pytezos
```
cd views_hangzhou/jsligo/test/pytezos
python3 -m unittest test_indice.py -v
```


## Contract Advisor

### Compile advisor contract 
- generates michelson code
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile contract views_hangzhou/jsligo/advisor.jsligo -e advisorMain --protocol hangzhou > views_hangzhou/jsligo/compiled/advisor.tz
```
- generates michelson code in JSON format
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile contract views_hangzhou/jsligo/advisor.jsligo --michelson-format json -e advisorMain --protocol hangzhou > views_hangzhou/jsligo/compiled/advisor.json
```

### Compile advisor storage

- With empty storage and trivial lambda function
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile storage views_hangzhou/jsligo/advisor.jsligo '{indiceAddress: ("KT1D99kSAsGuLNmT1CAZWx51vgvJpzSQuoZn" as address), algorithm: (i : int) : bool => { return false }, result: false}' -e advisorMain --protocol hangzhou
```

- With less trivial lambda function
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile storage views_hangzhou/jsligo/advisor.jsligo '{indiceAddress: ("KT1D99kSAsGuLNmT1CAZWx51vgvJpzSQuoZn" as address), algorithm: (i : int) : bool => { if (i < 10) { return true } else { return false } }, result: false}' -e advisorMain --protocol hangzhou
```

### Compile parameter (with ligo compiler) into Michelson expression

- For entrypoint ChangeAlgorithm
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile parameter views_hangzhou/jsligo/advisor.jsligo 'ChangeAlgorithm((i : int) : bool => { return false })' -e advisorMain --protocol hangzhou
```

- For entrypoint ExecuteAlgorithm
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile parameter views_hangzhou/jsligo/advisor.jsligo 'ExecuteAlgorithm(unit)' -e advisorMain --protocol hangzhou
```

### Simulate execution of entrypoints (with ligo compiler)

- For entrypoint ChangeAlgorithm
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next run dry-run views_hangzhou/jsligo/advisor.jsligo  'ChangeAlgorithm((i : int) : bool => { return false })' '{indiceAddress: ("KT1D99kSAsGuLNmT1CAZWx51vgvJpzSQuoZn" as address), algorithm: (i : int) : bool => { if (i < 10) { return true } else { return false } }, result: false}' -e advisorMain --protocol hangzhou
```

- For entrypoint ExecuteAlgorithm (fails due to on-chain views)
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next run dry-run views_hangzhou/jsligo/advisor.jsligo 'ExecuteAlgorithm(unit)' '{indiceAddress: ("KT1D99kSAsGuLNmT1CAZWx51vgvJpzSQuoZn" as address), algorithm: (i : int) : bool => { if (i < 10) { return true } else { return false } }, result: false}' -e advisorMain --protocol hangzhou
```

### Originate the Advisor contract with tezos-client CLI

#### Prepare initial storage 

- Compile the sotrage into Michelson expression
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next compile storage views_hangzhou/jsligo/advisor.jsligo '{indiceAddress: ("KT1D99kSAsGuLNmT1CAZWx51vgvJpzSQuoZn" as address), algorithm: (i : int) : bool => { if (i < 10) { return true } else { return false } }, result: false}' -e advisorMain --protocol hangzhou
```

This command produces the following Michelson storage:
```
(Pair (Pair { PUSH int 10 ;
              SWAP ;
              COMPARE ;
              LT ;
              IF { PUSH bool True } { PUSH bool False } }
            "KT1D99kSAsGuLNmT1CAZWx51vgvJpzSQuoZn")
      False)
```

- Deploy Advisor contract (with a sandbox)

```
tezos-client originate contract advisor transferring 1 from bootstrap1  running '/home/frank/Marigold/advisor/views_hangzhou/jsligo/compiled/advisor.tz' --init '(Pair (Pair { PUSH int 10 ; SWAP ;COMPARE ;LT ;IF { PUSH bool True } { PUSH bool False } } "KT1D99kSAsGuLNmT1CAZWx51vgvJpzSQuoZn") False)' --dry-run
```

- Verify the entrypoint is callable
```
tezos-client transfer 0 from bootstrap3 to advisor --arg '(Right Unit)' --dry-run
tezos-client transfer 0 from tz1RyejUffjfnHzWoRp1vYyZwGnfPuHsD5F5 to KT1N6thksQc11X8bgL9AUBHztcFVEPj9C1bL --entrypoint "executeAlgorithm" --arg "Unit"
```

### Test deployment/interact (with ligo compiler)
```
docker run --rm -v "$PWD":"$PWD" -w "$PWD" ligolang/ligo:next run test views_hangzhou/jsligo/test/ligo/test.jsligo --protocol hangzhou
```


### Unit test pytezos
```
cd views_hangzhou/jsligo/test/pytezos
python3 -m unittest test_advisor.py -v
```

### Deploy (with Taquito)
```
tsc deploy.ts --resolveJsonModule -esModuleInterop
node deploy.js
```