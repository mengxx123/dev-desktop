<template>
    <div id="wrapper">
        <h2>服务器</h2>
        <ui-raised-button label="按钮" />
        <div>
            <input v-model="host" />
        </div>
        <div>
            <input v-model="username"/>
        </div>
        <div>
            <input v-model="password"/>
        </div>
        <button @click="start">链接</button>
        <div v-if="isConnect">
            <input v-model="shell">
            <button @click="exec">执行</button>
            <div><pre><code>{{ output }}</code></pre>
            </div>
            <div v-if="isLs">
                <ul v-for="file in fileList">
                    {{ file }}
                </ul>

            </div>
        </div>
        <h2>数据库</h2>
        <router-link to="/database">数据库</router-link>

    </div>
</template>

<script>
    /* eslint-disable */
    import SystemInformation from './LandingPage/SystemInformation'
    var Client = require('ssh2').Client;
    var conn = new Client();

    export default {
        name: 'landing-page',
        components: {SystemInformation},
        data() {
            return {
                host: '120.78.177.9',
                port: 22,
                username: 'root',
                password: '',
                isConnect: false,
//                shell: 'uptime',
                shell: 'ls',
                output: '',
                isLs: true,
                fileList: []
            }
        },
        methods: {
            open (link) {
                this.$electron.shell.openExternal(link)
            },
            getFileList() {
                conn.sftp(function(err, sftp) {
                    if (err) throw err;
                    sftp.readdir('/root', function(err, list) {
                        if (err) throw err;
                        console.dir(list);
                        conn.end();
                    });
                });
            },
            start() {
                console.log('链接246')
                conn.on('ready', () => {
                    this.isConnect = true
                    console.log('连接上了！')
//                    this.getFileList()
                })
                    .on('end', function () {
                        console.log('完成')
                    })
                    .on('error', function(err) {
                        console.log('错误' + err)
                    })
                    .connect({
                        host: this.host,
                        port: 22,
                        username: this.username,
                        password: this.password
                    });
            },
            afterConnect(callback) {
                conn.on('ready', () => {
                    this.isConnect = true
                    console.log('连接上了！')
                    callback && callback()
                })
                    .on('end', function () {
                        console.log('完成')
                    })
                    .on('error', function(err) {
                        console.log('错误' + err)
                    })
                    .connect({
                        host: this.host,
                        port: 22,
                        username: this.username,
                        password: this.password
                    });
            },
            exec() {
                this.afterConnect(() => {
                    if (this.shell === 'ls') {
                        this.isLs = true
                    } else {
                        this.isLs = false
                    }
                    conn.exec(this.shell, (err, stream) => {
                        if (err) throw err
                        stream.on('close', function(code, signal) {
                            console.log('Stream :: close :: code: ' + code + ', signal: ' + signal);
                            conn.end();
                        }).on('data', data => {
                            console.log('STDOUT: ' + data);
                            this.output = '' + data
                            if (this.isLs) {
                                this.fileList = this.output.split('\n')
                            }
                        }).stderr.on('data', data => {
                            console.log('STDERR: ' + data);
                            this.output = '' + data
                        });
                    });
                })
            }
        }
    }
</script>

<style>
    @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');
</style>
