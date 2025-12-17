<script>
    const numF = new Intl.NumberFormat("en-US", {
        minimumFractionDigits: 3,
        maximumFractionDigits: 3,
    });
    const numFS = new Intl.NumberFormat("en-US", {
        signDisplay: "always",
        minimumFractionDigits: 3,
        maximumFractionDigits: 3,
    });
    const numI = new Intl.NumberFormat("en-US", {
        minimumIntegerDigits: 1,
        minimumFractionDigits: 0,
        maximumFractionDigits: 0,
    });
    const maxRoot = 10;
    let polar = $state({
        mag: 1,
        phase: angleMod(0.8),
    });

    let zoom = $state(0);
    let zoomFactor = $derived(Math.exp(zoom) * 100);

    let cartesian = $derived({
        re: Math.cos(polar.phase) * polar.mag,
        im: Math.sin(polar.phase) * polar.mag,
    });

    let root = $state(3);
    let selectedRoot = $state(0);
    let showSelectedRoot = $state(true);

    function angleMod(angle) {
        while (angle > Math.PI) {
            angle -= 2 * Math.PI;
        }
        while (angle < -Math.PI) {
            angle += 2 * Math.PI;
        }

        return angle;
    }

    let offsetPhase = $state(Math.PI);
    let cartesianOffset = $derived({
        re: Math.cos(offsetPhase) * Math.hypot(size.y, size.x),
        im: Math.sin(offsetPhase) * Math.hypot(size.y, size.x),
    });
    let cartesianOffsetMin = $derived({
        re: Math.cos(offsetPhase) * Math.min(size.y, size.x),
        im: Math.sin(offsetPhase) * Math.min(size.y, size.x),
    });

    let polarRoots = $derived(
        Array(root)
            .fill(root)
            .map((r, i) => ({
                mag: Math.pow(polar.mag, 1 / r),
                phase:
                    (angleMod(polar.phase - offsetPhase + Math.PI) +
                        offsetPhase -
                        Math.PI +
                        Math.PI * 2 * i) /
                    r,
            })),
    );

    let cartesianRoots = $derived(
        polarRoots.map((p) => ({
            re: Math.cos(p.phase) * p.mag,
            im: Math.sin(p.phase) * p.mag,
        })),
    );

    let size = $state({
        x: 100,
        y: 100,
    });

    function svgEventPosition(evt) {
        const svg = evt.currentTarget.ownerSVGElement;
        const pt = svg.createSVGPoint();
        pt.x = evt.clientX;
        pt.y = evt.clientY;
        const svgGlobal = pt.matrixTransform(svg.getScreenCTM().inverse());

        return { x: svgGlobal.x, y: svgGlobal.y };
    }

    let dragOffset = { x: 0, y: 0 };
    function dragStart(evt) {
        if (evt.isPrimary) {
            const svgGlobal = svgEventPosition(evt);
            dragOffset.x = evt.currentTarget.getAttribute("cx") - svgGlobal.x;
            dragOffset.y = evt.currentTarget.getAttribute("cy") - svgGlobal.y;
            evt.currentTarget.setPointerCapture(evt.pointerId);
        }
    }
    function dragMove(evt) {
        if (evt.currentTarget.hasPointerCapture(evt.pointerId)) {
            const svgGlobal = svgEventPosition(evt);
            svgGlobal.x += dragOffset.x;
            svgGlobal.y += dragOffset.y;
            polar.mag = Math.hypot(
                svgGlobal.y / zoomFactor,
                svgGlobal.x / zoomFactor,
            );
            polar.phase = Math.atan2(-svgGlobal.y, svgGlobal.x);
        }
    }
    function dragEnd() {}

    function dragRootStart(evt) {
        if (evt.isPrimary) {
            const svgGlobal = svgEventPosition(evt);
            dragOffset.x = evt.currentTarget.getAttribute("cx") - svgGlobal.x;
            dragOffset.y = evt.currentTarget.getAttribute("cy") - svgGlobal.y;
            evt.currentTarget.setPointerCapture(evt.pointerId);
        }
    }
    function dragRootMove(evt) {
        if (evt.currentTarget.hasPointerCapture(evt.pointerId)) {
            const svgGlobal = svgEventPosition(evt);
            svgGlobal.x += dragOffset.x;
            svgGlobal.y += dragOffset.y;
            polar.mag = Math.pow(
                Math.hypot(svgGlobal.y / zoomFactor, svgGlobal.x / zoomFactor),
                root,
            );
            polar.phase = angleMod(
                Math.atan2(-svgGlobal.y, svgGlobal.x) * root,
            );
        }
    }
    function dragRootEnd() {}

    function dragDiscStart(evt) {
        if (evt.isPrimary) {
            const svgGlobal = svgEventPosition(evt);
            dragOffset.x = evt.currentTarget.getAttribute("cx") - svgGlobal.x;
            dragOffset.y = evt.currentTarget.getAttribute("cy") - svgGlobal.y;
            evt.currentTarget.setPointerCapture(evt.pointerId);
        }
    }
    function dragDiscMove(evt) {
        if (evt.currentTarget.hasPointerCapture(evt.pointerId)) {
            const svgGlobal = svgEventPosition(evt);
            svgGlobal.x += dragOffset.x;
            svgGlobal.y += dragOffset.y;
            offsetPhase = angleMod(Math.atan2(-svgGlobal.y, svgGlobal.x));
        }
    }
    function dragDiscEnd() {}

    function resetOffset() {
        offsetPhase = Math.PI;
    }
    function resetZoom() {
        zoom = 0;
    }
    function resetMagnitude() {
        polar.mag = 1;
    }
    function resetPhase() {
        polar.phase = 0;
    }
</script>

<div class="fullscreen">
    <svg
        class="drawing"
        onwheel={(evt) => {
            zoom -= evt.deltaY / 5 / 120;
        }}
        bind:clientWidth={size.x}
        bind:clientHeight={size.y}
        viewBox="{-size.x / 2} {-size.y / 2} {size.x} {size.y}"
    >
        <circle
            cx={0}
            cy={0}
            r={zoomFactor}
            fill="none"
            stroke-width="2"
            stroke="#ccc"
            vector-effect="non-scaling-stroke"
            stroke-dasharray="3 3"
        ></circle>
        <circle cx={0} cy={0} r={2}></circle>
        <line
            x1={-size.x / 2}
            x2={size.x / 2}
            y1={0}
            y2={0}
            stroke="black"
            vector-effect="non-scaling-stroke"
        />
        <line
            y1={-size.y / 2}
            y2={size.y / 2}
            x1={0}
            x2={0}
            stroke="black"
            vector-effect="non-scaling-stroke"
        />
        <g class={{ hidden: !showSelectedRoot }}>
            <line
                x1={0}
                y1={0}
                x2={cartesianOffset.re * zoomFactor}
                y2={-cartesianOffset.im * zoomFactor}
                stroke-width="2"
                stroke="#000"
                vector-effect="non-scaling-stroke"
                stroke-dasharray="3 5"
            />

            <circle
                cx={cartesianOffsetMin.re / 3}
                cy={-cartesianOffsetMin.im / 3}
                r={5}
                fill="hsl({(-offsetPhase * 180) / Math.PI}, 100%, 50%)"
                stroke="#278285"
                stroke-width="2"
                cursor="grab"
                onpointerdown={dragDiscStart}
                onpointermove={dragDiscMove}
                onpointerup={dragDiscEnd}
            ></circle>
        </g>
        <line
            x1={-zoomFactor}
            x2={-zoomFactor}
            y1={10}
            y2={-10}
            stroke="black"
            vector-effect="non-scaling-stroke"
        />
        <line
            x1={zoomFactor}
            x2={zoomFactor}
            y1={10}
            y2={-10}
            stroke="black"
            vector-effect="non-scaling-stroke"
        />
        <line
            y1={-zoomFactor}
            y2={-zoomFactor}
            x1={10}
            x2={-10}
            stroke="black"
            vector-effect="non-scaling-stroke"
        />
        <line
            y1={zoomFactor}
            y2={zoomFactor}
            x1={10}
            x2={-10}
            stroke="black"
            vector-effect="non-scaling-stroke"
        />
        <text font-size="16" x={zoomFactor} y={28} text-anchor="middle">
            <tspan font-size="16">1</tspan>
        </text>
        <text font-size="16" x={-zoomFactor} y={28} text-anchor="middle">
            <tspan font-size="16">-1</tspan>
        </text>
        <text font-size="16" y={-zoomFactor + 5} x={-28} text-anchor="middle">
            <tspan font-size="16">j</tspan>
        </text>
        <text font-size="16" y={zoomFactor + 5} x={-28} text-anchor="middle">
            <tspan font-size="16">-j</tspan>
        </text>
        <text font-size="16" x={size.x / 2 - 32} y={-8}>
            <tspan font-size="16">Re</tspan>
        </text>
        <text font-size="16" y={-size.y / 2 + 16} x={4}>
            <tspan font-size="16">Im</tspan>
        </text>
        <circle
            cx={0}
            cy={0}
            r={polarRoots[0].mag * zoomFactor}
            fill="none"
            stroke-width="1"
            stroke="#a77"
            vector-effect="non-scaling-stroke"
            stroke-dasharray="1 5"
        ></circle>

        {#each cartesianRoots as car, c (c)}
            <line
                x1={0}
                y1={0}
                x2={car.re * zoomFactor}
                y2={-car.im * zoomFactor}
                stroke="black"
                vector-effect="non-scaling-stroke"
            />
            <circle
                cx={car.re * zoomFactor}
                cy={-car.im * zoomFactor}
                r={c != selectedRoot || !showSelectedRoot ? 8 : 10}
                fill={c != selectedRoot % root || !showSelectedRoot
                    ? "maroon"
                    : "#f40"}
                stroke="white"
                stroke-width="2"
                cursor="grab"
                onpointerdown={dragRootStart}
                onpointermove={dragRootMove}
                onpointerup={dragRootEnd}
            ></circle>
            <text
                font-size="16"
                class={{ hidden: !showSelectedRoot }}
                pointer-events="none"
                x={car.re * zoomFactor}
                y={-car.im * zoomFactor - 0.2 * zoomFactor}
            >
                <tspan font-size="16">r</tspan><tspan
                    baseline-shift="sub"
                    vertical-align="sub"
                    font-size="small">{c + 1}</tspan
                >
            </text>
        {/each}

        <line
            x1={0}
            y1={0}
            x2={cartesian.re * zoomFactor}
            y2={-cartesian.im * zoomFactor}
            stroke="black"
            vector-effect="non-scaling-stroke"
        />
        <circle
            cx={cartesian.re * zoomFactor}
            cy={-cartesian.im * zoomFactor}
            r={12}
            fill="purple"
            cursor="grab"
            stroke="white"
            stroke-width="2"
            onpointerdown={dragStart}
            onpointermove={dragMove}
            onpointerup={dragEnd}
        ></circle>
        <text
            font-size="16"
            pointer-events="none"
            x={cartesian.re * zoomFactor}
            y={-cartesian.im * zoomFactor - 0.2 * zoomFactor}
            fill="purple"
            cursor="grab"
        >
            <tspan font-size="16">z</tspan><tspan
                baseline-shift="sub"
                vertical-align="sub"
                font-size="small">0</tspan
            >
            <tspan font-size="16">[</tspan>
            <tspan font-size="16">Magnitude: {numF.format(polar.mag)}</tspan>
            <tspan font-size="16"
                >Phase: {numF.format(polar.phase / Math.PI)}&pi;</tspan
            >
            <tspan font-size="16">]</tspan>
        </text>
    </svg>
    <div class="options">
        <h1>Complex Roots</h1>
        <div class="controls">
            <div class="slider-control">
                <label
                    ><span>Zoom</span>
                    <input
                        type="range"
                        bind:value={zoom}
                        min="-3"
                        max="3"
                        step={0.1}
                    /></label
                >
                <button
                    type="button"
                    class="reset-button"
                    tabindex="-1"
                    onkeypress={resetZoom}
                    onclick={resetZoom}>&times;</button
                >
                <output>{numFS.format(zoom)}</output>
            </div>
            <div class="slider-control" style:accent-color="purple">
                <label
                    ><span>Magnitude</span>
                    <input
                        type="range"
                        bind:value={polar.mag}
                        min={0.1 / zoomFactor}
                        max={500 / zoomFactor}
                        step={0.01 / zoomFactor}
                    /></label
                >
                <button
                    type="button"
                    class="reset-button"
                    tabindex="-1"
                    onkeypress={resetMagnitude}
                    onclick={resetMagnitude}>&times;</button
                >

                <output>{numF.format(polar.mag)}</output>
            </div>
            <div class="slider-control" style:accent-color="purple">
                <label
                    ><span>Phase</span>
                    <input
                        type="range"
                        bind:value={polar.phase}
                        min={numF.format(-Math.PI)}
                        max={numF.format(Math.PI)}
                        step={0.01}
                    /></label
                >
                <button
                    type="button"
                    class="reset-button"
                    tabindex="-1"
                    onkeypress={resetPhase}
                    onclick={resetPhase}>&times;</button
                >

                <output>{numF.format(polar.phase / Math.PI)} &pi;</output>
            </div>
            <div class="slider-control" style:accent-color="maroon">
                <label
                    ><span>n<sup>th</sup>Roots:</span>
                    <input
                        type="range"
                        bind:value={root}
                        min="1"
                        max={maxRoot}
                    /></label
                ><span></span><output>{numI.format(root)}</output>
            </div>

            <div class="slider-control">
                <div>
                    <label>
                        <span>Show Designated Root</span>
                        <input
                            style:accent-color="#f20"
                            type="checkbox"
                            bind:checked={showSelectedRoot}
                        />
                    </label>
                </div>
            </div>
            <div class="slider-control" style:accent-color="#f20">
                <label
                    ><span>Designated Root:</span>
                    <input
                        disabled={!showSelectedRoot}
                        type="range"
                        bind:value={selectedRoot}
                        min="0"
                        max={maxRoot}
                    /></label
                ><span></span><output>{numI.format(selectedRoot)}</output>
            </div>
            <div class="slider-control" style:accent-color="#278285">
                <label
                    ><span>Discontinuity:</span>
                    <input
                        disabled={!showSelectedRoot}
                        type="range"
                        bind:value={offsetPhase}
                        min={-Math.PI}
                        max={Math.PI}
                        step={0.01}
                    /></label
                >
                <button
                    disabled={!showSelectedRoot}
                    type="button"
                    class="reset-button"
                    tabindex="-1"
                    onkeypress={resetOffset}
                    onclick={resetOffset}>&times;</button
                >
                <output>{numF.format(offsetPhase / Math.PI)} &pi;</output>
            </div>
        </div>
        <p>
            <a target="_blank" href="//tools.laszlokorte.de"
                >More edcuational tools</a
            >
        </p>
        <details>
            <summary>Explanation</summary>
            <div>
                <p>
                    Taking a complex number <code>z</code> to an integer power
                    <code>n</code>
                    multiplies its phase (angle) &phi; by <code>n</code> and
                    exponentiates its magnitude to the power of <code>n</code>.
                </p>
                <p>
                    Due to the cyclical nature of rotation there exist multiple
                    angles that when multiplied by a natural number <code
                        >n</code
                    > yield the same final rotation.
                </p>
                <p>
                    For example turning by 0° 4 times in a row lands you at a
                    final orientation of 0°. But turning 4 times by 90° lands
                    you at 0° as well (a full turn), and 4 times 180° lands you
                    also at 0° (after two full rotations). Finally turning by
                    270° four times will also result in an 0° orientation (a
                    total of 3 full turns).
                </p>
                <p>
                    There a infinitely more angles that also make you end up at
                    0° after 4 turns. For example 4 times 360°, but 360° on its
                    own is no different from 0° anyway. Notice that the angles
                    0°, 90°, 180° and 270° different from each other on their
                    own but 4 each of them resuls in a 0° orientitation after 4
                    turns.
                </p>
                <p>
                    In general for each natural number <code>n</code> there are
                    <code>n</code> <em>different</em> angles that when
                    multiplied by <code>n</code> will turn you back to 0°.
                </p>
                <p>
                    In context of complex numbers this means that there are <code
                        >n</code
                    >
                    different complex numbers that, when raised to the power of
                    <code>n</code>, will yield the same result.
                </p>
                <p>
                    Conversely this means that for any complex number <code
                        >z</code
                    >
                    there are <code>n</code> solutions for the <code>n</code>-th
                    root of
                    <code>z</code>.
                </p>
                <p>
                    The plot shows a complex number <code>z<sub>0</sub></code>
                    (big purple dot) and all <code>n</code>
                    of its <code>n-th</code> roots (<code>r<sub>1</sub></code>
                    to <code>r<sub>n</sub></code>).
                </p>
                <p>
                    You can drag the complex number (z<sub>0</sub>) around and
                    observe how the roots (r<sub>1</sub>, r<sub>2</sub>,
                    &hellip;) move around accordingly. Notice that all the roots
                    have the same magnitude (the n-th root of the complex
                    numbers real-valued magnitude)
                </p>
                <p>
                    You can also drag any of the roots around and see the
                    complex number <code>z<sub>0</sub></code> change accordingly.
                    Keep in mind that all the n roots have the same n-th power so
                    it does not matter which root you drag around.
                </p>
                <p>
                    One of the roots is highlighted in a brighter red color.
                    This is the <em>designated root</em>. This is the single
                    number you would want to get as result when applying the
                    root function in a calculation. For example when taking the
                    square-root of <code>25 </code> you have learned that both
                    <code>-5</code>
                    and <code>+5</code> are valid solutions but you still would expect
                    that one of them is the actual number that a calculator would
                    give you (usually the positive number +5).
                </p>
                <p>
                    But with complex numbers as soon as there are more than just
                    one positive and one negative real number, how would you
                    deterministically decide <em>which</em> one to pick as the
                    single <em>designated</em> solution?
                </p>
                <p>
                    By using the <em>Designated Root</em> slider you can pick one
                    of the roots you like best. Try to configure it to always give
                    you the sensible solution, for example always the one closest
                    to the positive real axis.
                </p>
                <p>
                    After making your choice try to move the original complex
                    number around, making one full turn around the unit circle.
                </p>
                <p>
                    Notice that at some point the designated root jumps around.
                    This is because another of the roots is now closer to the
                    location your originally picked as designated root.
                </p>
                <p>
                    The point (i.e. angle) at which this jump happens is called
                    a discontinuity. You can choose where is discontinuity shall
                    happen but as long as you assume one of the roots to be
                    marked as designated root the jump will (must) happen at one
                    point.
                </p>
                <p>
                    <strong
                        >In short: We can see that as long as we assume (or
                        declare) one of the roots as special the root function
                        overall can not be continious.</strong
                    >
                </p>
                <p>
                    Now lets try to get rid of the assumtion of having one
                    <em>designated root</em>. Uncheck the
                    <label
                        style="display: inline flex; align-items:center; background-color:  #5555; padding: 0.5ex; border-radius: 4px;"
                    >
                        <input
                            type="checkbox"
                            bind:checked={showSelectedRoot}
                        />
                        <span>Show Designated Root Checkbox</span>
                    </label>
                </p>
                <p>
                    Now observe that when moving the complex number <code
                        >z</code
                    > around no jumps are noticable anymore since you can no longer
                    discern the different roots.
                </p>
                <p>
                    Also notice that it takes multiple turns of the original
                    complex number around the origin for the the set of roots to
                    take a full turn. This is kind of an illusion caused by <em
                        >not labeling</em
                    > the individual roots but looking at all of them together as
                    a set.
                </p>
                <p>
                    As soon as we attach labels to one or more of the roots, as
                    we tried with the <em>designated root</em>, the illusion
                    disappears but instead we can observe the discontinuities.
                </p>
                <p>
                    Imagine how sticking a posted note onto a <a
                        href="https://en.wikipedia.org/wiki/Barber%27s_pole"
                        target="_blank">Barber's pole</a
                    > would destroy the illusion.
                </p>
                <p>
                    Now observe how it takes exactly n full rotation of the
                    complex number <code>z</code> around the origin until a
                    single root takes <em>one full continious rotation</em> around
                    the origin.
                </p>
                <p>
                    But wait! Think about what it <em>actually means</em> to say "one
                    of roots to takes a full rotation" if we just claimed that we
                    do not label the individual roots at all and therefor can not
                    discern them.
                </p>
            </div>
        </details>
    </div>
</div>

<style>
    text {
        font-size: 1em;
    }
    summary {
        cursor: pointer;
        background-color: #0001;
        border-bottom: 1px solid #0003;
        padding: 1ex;
    }
    details {
        max-width: 100%;
        background-color: #fff;
    }
    details > :nth-child(2) {
        padding: 1em;
    }
    h1 {
        margin: 0;
        white-space: nowrap;
    }
    .options {
        padding: 1em;
        background: #fffa;
        margin: 1ex;
        border: 2px solid #aaa;
        align-self: start;
        justify-self: start;
        max-height: 90vh;
        overflow: auto;
    }
    .controls {
        display: grid;
        grid-template-columns: auto auto auto auto;
        grid-template-rows: auto;
        align-items: center;
        gap: 0.5ex;
    }
    .slider-control {
        display: grid;
        grid-template-columns: auto 2fr auto;
        gap: 1em;
        display: subgrid;
        grid-column: 1 / 4;
        align-items: center;
    }
    .slider-control > label {
        gap: 1em;
        display: grid;
        grid-template-columns: minmax(7em, auto) 1fr;
        display: subgrid;
        grid-column: 1 / 2;
        align-items: center;
    }

    .drawing {
        display: block;
        width: 100%;
        height: 100%;
        user-select: none;
    }
    .fullscreen {
        color: #000;
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        width: 100%;
        height: 100%;
        display: grid;
        justify-content: stretch;
        align-content: stretch;
        background-color: #fff;
        background-size: contain;
        background-image: conic-gradient(
            from 90deg,
            rgba(255, 0, 0, 0.2),
            rgba(255, 0, 255, 0.2),
            rgba(0, 0, 255, 0.2),
            rgba(0, 255, 255, 0.2),
            rgba(0, 255, 0, 0.2),
            rgba(255, 255, 0, 0.2),
            rgba(255, 0, 0, 0.2)
        );
    }

    .fullscreen > * {
        grid-area: 1 / 1 / 2 / 2;
    }

    .hidden {
        display: none;
    }
    .reset-button {
        background-color: black;
        color: #fff;
        border: none;
        width: 1.5em;
        height: 1.5em;
        vertical-align: middle;
        font-family: monospace;
        padding: 0;
        box-sizing: border-box;
        display: flex;
        align-items: center;
        justify-content: center;
        border-radius: 5px;
    }
    a {
        color: purple;
    }

    input[type="range"] {
        width: 15em;
    }
    output {
        width: 5em;
    }
    a {
        color: royalblue;
    }
</style>
