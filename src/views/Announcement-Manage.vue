<template>
    <div v-if="editMode === 'create' || editMode === 'update'">
        <el-form :model="announcement" :rules="rules" ref="announcement">
            <el-form-item prop="title">
                <el-input type="text" v-model="announcement.title" placeholder="标题" maxlength="100" show-word-limit/>
            </el-form-item>
            <el-form-item prop="content">
                <el-input type="textarea" v-model="announcement.content" :autosize="{minRows: 6}"
                          placeholder="内容" minlength="10" maxlength="2000" show-word-limit/>
            </el-form-item>
            <el-form-item class="text-right">
                <el-button size="small" @click="onSubmit('announcement')" type="primary" :loading="loading">
                    确认
                </el-button>
                <el-button size="small" @click="editMode = ''">取消</el-button>
            </el-form-item>
        </el-form>
    </div>
    <div v-else>
        <el-table ref="table" :data="announcements" tooltipEffect="light" style="width: 100%" border>
            <el-table-column prop="title" label="标题" align="center" width="600" show-overflow-tooltip/>
            <el-table-column prop="createTime" label="创建时间" align="center" />
            <el-table-column prop="updateTime" label="修改时间" align="center"/>
            <el-table-column label="操作" align="center" width="100px">
                <template #header #default="scope">
                    <el-button type="primary" size="mini" @click="createAnnouncement">新增</el-button>
                </template>
                <template #default="scope">
                    <el-dropdown @command="handleCommand($event,scope.row)" trigger="click">
                        <span class="el-dropdown-link">
                            <i class="el-icon-s-operation"></i>
                        </span>
                        <template #dropdown>
                            <el-dropdown-menu>
                                <el-dropdown-item command="updateAnnouncement">编辑</el-dropdown-item>
                                <el-dropdown-item command="deleteAnnouncement">删除</el-dropdown-item>
                            </el-dropdown-menu>
                        </template>
                    </el-dropdown>
                </template>
            </el-table-column>
        </el-table>
        <div class="pagination">
            <el-pagination background layout="prev, pager, next" :pager-count="5" :total="size"
                           :hide-on-single-page="true" @current-change="handlePageChange">
            </el-pagination>
        </div>
    </div>
</template>

<script>
import {createAnnouncement, deleteAnnouncement, getAnnouncements, updateAnnouncement} from '/@/utils/api'

export default {
    name: "Announcement-Manage",
    data() {
        return {
            announcement: {},
            announcements: [],
            size: 0,
            editMode: '',
            loading: false,
            rules: {
                title: [
                    {required: true, message: '请输入标题', trigger: 'blur'},
                    {min: 5, max: 50, message: '长度在 5 到 50 个字符', trigger: 'blur'}
                ],
                content: [
                    {required: true, message: '请输入内容', trigger: 'blur'},
                    {min: 5, max: 2000, message: '长度在 5 到 2000 个字符', trigger: 'blur'}
                ]
            }
        }
    },
    created() {
        this.getAnnouncements()
    },
    methods: {
        getAnnouncements(pageNumber) {
            getAnnouncements({pageNumber}).then(result => {
                if (result.code === '0000') {
                    this.announcements = result.data.list
                    this.size = result.data.size
                    this.$refs.table.doLayout()
                }
            })
        },
        onSubmit(announcement) {
            this.$refs[announcement].validate((valid) => {
                if (valid) {
                    this.loading = true
                    if (this.editMode === 'create') {
                        createAnnouncement(this.announcement).then(result => {
                            if (result.code === '0000') {
                                this.$refs[announcement].resetFields()
                                this.$message.success("发布成功！")
                            }
                        }).finally(() => this.loading = false)
                    }
                    if (this.editMode === 'update') {
                        updateAnnouncement(this.announcement).then(result => {
                            if (result.code === '0000') {
                                this.editMode = ''
                                this.$message.success("更新成功！")
                            }
                        }).finally(() => this.loading = false)
                    }
                }
            })
        },
        createAnnouncement() {
            this.editMode = 'create'
            this.announcement = {}
        },
        updateAnnouncement(row) {
            this.editMode = 'update'
            this.announcement = row
        },
        deleteAnnouncement(row) {
            this.$confirm("确定删除？").then(() => {
                deleteAnnouncement(row.id).then(result => {
                    if (result.code === '0000') {
                        let index = this.announcements.indexOf(row)
                        this.announcements.splice(index, 1)
                        this.$message.success("删除成功！")
                    }
                })
            }).catch(() => {
                this.$message.warning("已取消！")
            })
        },
        handleCommand(command, row) {
            switch (command) {
                case "updateAnnouncement":
                    this.updateAnnouncement(row)
                    break
                case "deleteAnnouncement":
                    this.deleteAnnouncement(row)
                    break
            }
        },
        handlePageChange(pageNum) {
            this.getAnnouncements(pageNum)
        }
    }
}
</script>