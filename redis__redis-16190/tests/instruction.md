# Issue

**Title:** Fix undefined shifts in lua_cmsgpack int decoding

## Description

`mp_decode_to_lua_type()` shifted signed values in the `0xd2` (int 32) and
`0xd3` (int 64) decoders: for any high bit set (any negative number) the
intermediate result is not representable in the signed type — undefined
behavior. Reachable from every Lua script via
`EVAL "return cmsgpack.unpack(...)"`.

```bash
redis-cli EVAL 'return cmsgpack.unpack(string.char(210,128,0,0,0))' 0
```

```text
lua_cmsgpack.c:631:31: runtime error: left shift of 128 by 24 places cannot be represented in type 'int32_t' (aka 'int')
```

Found by Linux Verification Center (linuxtesting.org) with SVACE

<!-- CURSOR_SUMMARY -->
---

> [!NOTE]
> **Low Risk**
> Targeted decode-path fix with added regression tests; low blast radius beyond `cmsgpack` unpacking in Lua scripting.
>
> **Overview**
> **What changed**
> - Fixes undefined-behavior in `cmsgpack` decoding for signed integers (`0xd2` int32, `0xd3` int64) by assembling bytes in unsigned arithmetic, then casting to the signed result. This prevents illegal left-shifts on negative-promoted bytes.
> - Adds a regression test in `tests/unit/scripting.tcl` unpacking boundary negative int32/int64 payloads to assert correct values.
>
> **Impact**
> - Behavior for negative values should now match spec and be stable across compilers/sanitizers; no intentional API changes beyond correctness.
>
> <sup>Reviewed by [Cursor Bugbot](https://cursor.com/bugbot) for commit b180f10a68aef0fa584365f1606cbda203b956a0. Bugbot is set up for automated code reviews on this repo. Configure [here](https://www.cursor.com/dashboard/bugbot).</sup>
<!-- /CURSOR_SUMMARY -->

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `5b69ba2ce91f`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.