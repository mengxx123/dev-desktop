<template>
    <div id="wrapper">
        <router-link to="/">返回</router-link>
        <h2>数据库</h2>
        <ul>
            <li v-for="table in tables">{{ table }}</li>
        </ul>
    </div>
</template>

<script>
    /* eslint-disable */
    const mysql = require('mysql2/promise');

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
                fileList: [],
                tables: []
            }
        },
        mounted() {
            this.init()
        },
        methods: {
            async init() {
                console.log(1)
                // create the connection to database
                const connection = await mysql.createConnection({
                    host: 'localhost',
                    user: 'root',
                    database: 'ys_erp',
                    password: '123456'
                });
                let pool = mysql.createPool({database: 'ys_erp'});
                let rows = await connection.execute('show tables')
                console.log('哈哈')
                console.log('rows')
                rows = rows[0]
                console.log(rows)
                let tables = []
                for (let row of rows) {
                    tables.push(row['Tables_in_ys_erp'])
                }
                console.log(tables)
                this.tables = tables
                    // rows: [ { result: 12 } ]
                    // internally 'select 1 + ? + ? as result' is prepared first. On subsequent calls cached statement is re-used
                this.dealTable()
            },
            async dealTable() {
                for (let tables of this.tables) {
                    tables.push(row['Tables_in_ys_erp'])
                }
                let rows = await connection.execute('show tables')
            }
        }
    }
</script>

<style>
    @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');
</style>
