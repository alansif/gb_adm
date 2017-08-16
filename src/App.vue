<template>
    <div id="app" style="width:100%;padding:10px 0;position:relative;">
        <el-button type="primary" size="small" @click="refreshTable" style="margin-left:30px">刷新</el-button>
        <el-pagination style="display:inline-block;vertical-align:middle;"
                       @size-change="handleSizeChange"
                       @current-change="handleCurrentChange"
                       :current-page.sync="currentPage"
                       :page-sizes="[10, 25, 50]"
                       :page-size="pagesize"
                       layout="total, sizes, prev, pager, next"
                       :total="total">
        </el-pagination>
        <div style="padding:10px 30px">
            <el-table :data="tableData" stripe>
                <el-table-column prop="time" label="时间" :width="170" sortable>
                    <template scope="scope">
                        {{getTime(scope.row.time)}}
                    </template>
                </el-table-column>
                <el-table-column prop="remoteip" label="IP" :width="130" sortable/>
                <el-table-column prop="username" label="姓名" :width="100" sortable/>
                <el-table-column prop="phone" label="电话" :width="130" sortable/>
                <el-table-column prop="category" label="类别" :width="100" sortable/>
                <el-table-column prop="message" label="内容"/>
                <el-table-column prop="state" label="状态" :width="140" sortable>
                    <template scope="scope">
                        {{scope.row.state}}
                        <el-switch v-model="scope.row.state" on-text="" off-text="" on-value="已处理" off-value="未处理" @change="handleStateChange(scope.$index, scope.row, $event)"/>
                    </template>
                </el-table-column>
                <el-table-column label="备注" :width="70" align="center">
                    <template scope="scope">
                        <el-button @click="handleComment(scope.row)" icon="edit" type="text" size="small"></el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <el-dialog title="备注" :visible.sync="dialogVisible" size="tiny">
            <el-input type="textarea" :rows="6" v-model="activeRow.comment"></el-input>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取消</el-button>
                <el-button type="primary" @click="submitComment">确定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    import axios from 'axios'
    import moment from 'moment'

    export default {
        data() {
            return {
                tableData: [],
                currentPage: 1,
                pagesize: 10,
                total: 0,
                dialogVisible: false,
                activeRow:{}
            }
        },
        methods: {
            refreshTable() {
                this.currentPage = 1;
                axios({
                    method: 'get',
                    url: 'http://www.advahealth.com.cn:3939/api/v1/guestbook/be',
                    params:{
                        pagesize:this.pagesize,
                        pageindex:this.currentPage
                    }
                }).then(response => {
                    this.tableData = response.data.data;
                    this.total = response.data.summary.total;
                }).catch(error => {
                    console.log(error);
                });
            },
            getTime(v) {
                return moment(v).format('YYYY-MM-DD HH:mm:ss');
            },
            handleSizeChange(val) {
                this.pagesize = val;
                this.refreshTable();
            },
            handleCurrentChange(val) {
                this.refreshTable();
            },
            handleStateChange(index, row, evt) {
                axios({
                    method: 'post',
                    url: `http://www.advahealth.com.cn:3939/api/v1/guestbook/be/${row.uid}/state`,
                    data: { value: evt }
                }).then(response => {
                }).catch(error => {
                    console.log(error);
                });
            },
            handleComment(row) {
                this.activeRow = row;
                this.dialogVisible = true;
            },
            submitComment() {
                axios({
                    method: 'post',
                    url: `http://www.advahealth.com.cn:3939/api/v1/guestbook/be/${this.activeRow.uid}/comment`,
                    data: { value: this.activeRow.comment }
                }).then(response => {
                }).catch(error => {
                    console.log(error);
                });
                this.dialogVisible = false;
            }
        },
        mounted() {
            this.refreshTable();
        }
    }
</script>

<style>
    body {
        font-family: Helvetica, sans-serif;
    }
</style>
