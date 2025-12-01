(function(Scratch) {
    "use strict";

    class MicRecorderExtension {
        constructor() {
            this.stream = null;
            this.context = null;
            this.analyser = null;
            this.dataArray = null;
            this.recording = false;
            this.samples = [];
            this.lastRecording = [];
        }

        async _initAudio() {
            if (this.stream) return;
            this.stream = await navigator.mediaDevices.getUserMedia({ audio: true });

            this.context = new (window.AudioContext || window.webkitAudioContext)();
            const source = this.context.createMediaStreamSource(this.stream);

            this.analyser = this.context.createAnalyser();
            this.analyser.fftSize = 2048;

            this.dataArray = new Float32Array(this.analyser.fftSize);

            source.connect(this.analyser);
        }

        async startRecording() {
            await this._initAudio();
            this.recording = true;
            this.samples = [];
            this._recordLoop();
        }

        _recordLoop() {
            if (!this.recording) return;

            this.analyser.getFloatTimeDomainData(this.dataArray);

            for (let i = 0; i < this.dataArray.length; i += 16) {
                this.samples.push(this.dataArray[i]);
            }

            requestAnimationFrame(() => this._recordLoop());
        }

        stopRecording() {
            this.recording = false;
            this.lastRecording = [...this.samples];
        }

        micLevel() {
            if (!this.dataArray) return 0;
            let sum = 0;
            for (let i = 0; i < this.dataArray.length; i++) {
                sum += Math.abs(this.dataArray[i]);
            }
            return sum / this.dataArray.length;
        }

        saveToList(list) {
            if (!this.lastRecording.length) return;
            list.length = 0;
            for (const s of this.lastRecording) {
                list.push(String(s));
            }
        }

        playFromList(list) {
            if (!list.length) return;
            if (!this.context) {
                this.context = new (window.AudioContext || window.webkitAudioContext)();
            }

            const buffer = this.context.createBuffer(1, list.length, this.context.sampleRate);
            const data = buffer.getChannelData(0);

            for (let i = 0; i < list.length; i++) {
                data[i] = parseFloat(list[i]) || 0;
            }

            const src = this.context.createBufferSource();
            src.buffer = buffer;
            src.connect(this.context.destination);
            src.start();
        }

        sampleCount() {
            return this.lastRecording.length;
        }
    }

    Scratch.extensions.register(new MicRecorderExtension());
})(Scratch);
