<template>
    <div class="tags" v-show="repository.starred_at">
        <el-select v-model="selected" multiple filterable allow-create default-first-option placeholder="add tags"
                   @change="tagsAndRepositories" no-match-text="no match data" no-data-text="no data">
            <el-option v-for="tag in tags" :key="tag.id" :value="tag.id" :label="tag.name"></el-option>
        </el-select>
    </div>
</template>

<script>
    export default {
        name: "Tags",

        props: {
            repository: {
                type: Object,
                required: true,
            }
        },

        data() {
            return {
                tags: db.get('tags').value(),
                selected: [],
            }
        },

        methods: {
            // 更新标签和项目关联
            tagsAndRepositories(tags) {
                // 清空关联表
                let tagsAndRepositories = db.get('tagsAndRepositories');
                tagsAndRepositories.remove({repositoryId: this.repository.repo.id}).write();

                // 遍历当前项目标签，处理标签表和关联表
                tags.forEach(item => {
                    // 关联表的标签 ID
                    let tagId = '';

                    // 当前用户所有的标签数据
                    let tagIds = db.get('tags').map('id').value();
                    let tagNames = db.get('tags').map('name').value();

                    // 处理新添加标签，更新标签表、标签项目关联表
                    if (_.indexOf(tagIds, item) === -1) {
                        if (_.indexOf(tagNames, item) === -1) {
                            // 标签集合处理：如果新添加标签不在标签集合里则添加进集合
                            tagId = shortid.generate();
                            this.tags.push({id: tagId, name: item});
                        } else {
                            tagId = db.get('tags').find({name: item}).value().id;
                        }
                    } else {
                        tagId = item;
                    }

                    // 写入关联数据
                    let relationTagIds = tagsAndRepositories.filter({repositoryId: this.repository.repo.id}).map('tagId').value();
                    if (_.indexOf(relationTagIds, tagId) === -1) {
                        let data = {
                            tagId: tagId,
                            repositoryId: this.repository.repo.id
                        };
                        tagsAndRepositories.push(data).write();
                    }
                });
            },
        },

        watch: {
            // 监听到项目变化时，显示项目已打过的标签
            repository: function (n, o) {
                this.selected = db.get('tagsAndRepositories').filter({repositoryId: this.repository.repo.id}).map('tagId').value();
            }
        },
    }
</script>

<style scoped lang="scss">
    .tags {
        margin-top: 20px;

        .el-select {
            width: 100%;
        }
    }
</style>