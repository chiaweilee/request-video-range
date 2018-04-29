<template>
    <div @click="test()">
        result: {{result}}
    </div>
</template>

<script>
    const request = function (uri, piece) {
        if (typeof uri !== 'string' || typeof piece !== 'number') return
        const result = {
            header: undefined,
            response: []
        }
        const requestRange = function (uri, {start, end}) {
            const xhr = new XMLHttpRequest()
            // response callback
            xhr.onreadystatechange = function () {
                if (!result.header && xhr.readyState === 2) {
                    // response header
                    const max = parseInt(/0-1\/([0-9]+)/.exec(xhr.getResponseHeader('Content-Range'))[1])
                    const version = /([0-9]+\.[0-9]+\.[0-9]+)/.exec(xhr.getResponseHeader('X-Mod-H264-Streaming'))[1]
                    const bit = Math.floor(max / piece)
                    result['header'] = {
                        max,
                        version
                    }
                    xhr.abort()
                    // continue to request more
                    for (let i = 0; i <= piece; i++) {
                        requestRange(uri, {
                            start: i * bit,
                            end: (i + 1) * bit
                        })
                    }
                }
                // else
                // do not use 'elseif' here
                if (xhr.readyState === 3 && xhr.status === 206) {
                    result['response'].push({
                        response: xhr.response,
                        length: xhr.response.length
                    })
                    xhr.abort()
                }
            }
            // send request
            xhr.open('GET', uri)
            xhr.setRequestHeader('Range', 'bytes=' + start + '-' + end)
            xhr.send()
        }
        // request start
        requestRange(uri, {
            start: 0,
            end: 1
        })
        // return result directly
        // use data driven to render it
        return result
    }

    export default {
        data () {
            return {
                result: {}
            }
        },
        props: ['src'],
        mounted () {
            this.result = request(this.src, 3)
        },
        watch: {
            src () {
                this.$forceUpdate()
            }
        },
        methods: {
            test () {
                console.log(this.result)
            }
        }
    }
</script>
