<template>
    <div>
        <a-table class="AndTable" rowKey="id"
            :row-selection="{ selectedRowKeys: state.selectedRowKeys, onChange: onSelectChange }" :columns="columns"
            :data-source="thingList" :pagination="false">
            <template #bodyCell="{ column, record, text }">
                <template v-if="column.key == 1">
                    <img :src="record.cover" />
                </template>
                <template v-if="column.key == 2">
                    {{ record.title }}
                </template>
                <template v-if="column.key == 3">
                    {{ text }}元
                </template>
                <template v-if="column.key == 4">
                    <div class="inpNumBox">
                        <a-input-number class="inpNum" :min="1" :max="10" v-model:value="record.count" />
                        <div class="title">限购10件</div>
                    </div>

                </template>
                <template v-if="column.key == 5">
                    <div class="del" @click="delWith(record.id)">
                        <DeleteOutlined style="color: #f38181;font-size:20px;" />
                        <p class="delText">删除</p>
                    </div>

                </template>
            </template>
        </a-table>
        <div style="margin-bottom: 16px">
            <a-button type="primary" :disabled="!hasSelected" :loading="state.loading" @click="start">
                购买
            </a-button>
            <span style="margin-left: 8px">
                <template v-if="hasSelected">
                    {{ `选择 ${state.selectedRowKeys.length} 类商品 ${countNum} 件` }}
                </template>
            </span>
        </div>
    </div>
</template>
<script lang="ts" setup>
//@ts-ignore
import { computed, reactive, defineProps, PropType, defineEmits } from 'vue';
//@ts-ignore  使用Pinia
import { useUserStore } from '/@/store';
//@ts-ignore
import { removeWishUserApi } from '/@/api/index/thing'
import { DeleteOutlined } from "@ant-design/icons-vue"
import { number } from 'echarts';
//创建pinia对象
const userStore = useUserStore()
//emit子传父
const emit = defineEmits(['getWithList'])
//购物车数据接口定义
interface IthingList {
    classification: number;
    classification_title: string;
    collect_count: number;
    cover: string;
    create_time: string;
    id: number;
    price: string;
    pv: number;
    recommend_count: number;
    repertory: number;
    score: number;
    status: string;
    tag: number[];
    title: string;
    wish_count: number;
    count: number;
}

const props = defineProps({
    thingList: {
        type: Array as PropType<IthingList[]>,
        default: []
    }
})

const { thingList } = toRefs(props)
console.log(thingList.value)

var countNum = ref<number>(thingList.value.reduce((sum, item) => item.pv + sum, 0))

console.log(countNum);

type Key = string | number;

interface DataType {
    key: Key;
    thingName: string;
    age: number;
    address: string;
}
//约束表头类型
interface Icolumns {
    key?: number;
    align?: string;
    title: string;
    dataIndex: string;
    width?: number
}
//：Icolumns[]约束此变量只能是包裹Icolumns的数组
//表头元素
const columns: Icolumns[] = [
    {
        key: 1,
        title: '图片',
        dataIndex: 'thingName',
        align: 'center'
    },
    {
        key: 2,
        title: '外设名称',
        dataIndex: 'thingName',
        align: 'left'
    },
    {
        key: 3,
        title: '价格',
        dataIndex: 'price',
    },
    {
        key: 4,
        title: '数量',
        dataIndex: 'count',
        width: 100,
    },
    {
        key: 5,
        title: '操作',
        dataIndex: 'caoZuo',
    },
];

const state = reactive<{
    selectedRowKeys: Key[];
    loading: boolean;
}>({
    selectedRowKeys: [], // Check here to configure the default column
    loading: false,
});
const hasSelected = computed(() => state.selectedRowKeys.length > 0);

const start = () => {
    state.loading = true;
    // ajax request after empty completing
    setTimeout(() => {
        state.loading = false;
        state.selectedRowKeys = [];
    }, 1000);
};
//表单选中时触发事件
const onSelectChange = (selectedRowKeys: Key[]) => {
    console.log('selectedRowKeys changed: ', selectedRowKeys);
    state.selectedRowKeys = selectedRowKeys;
};
//外设数量改变时触发
const countChange = (number: number, itemId: number) => {
    console.log("countChange++++++++++++++++", number, itemId);
    // let thing: IthingList = thingList.find(item => item.id == itemId)
    console.log(thingList.value.find(item => item.id == itemId))

    // thing.pv = number
}

// 删除购物车 形参携带商品ID
const delWith = (id: number): void => {
    //从pinia中拿去用户name
    const userName = (userStore.user_name)
    //调用API接口 传参为：username: username, thingId: record.id
    removeWishUserApi({ username: userName, thingId: id }).then(res => {
        console.log(res)
        emit('getWithList')
        console.log(thingList.value);

    })

}
</script>
<style lang="less" scoped>
.AndTable {
    img {
        width: 85px;
        height: 85px;
        //图片不变形
        object-fit: cover;
    }

    .inpNumBox {
        display: flex;
        flex-wrap: nowrap;
        justify-content: center;
        align-items: center;
        flex-direction: column;

        .inpNum {
            width: 100%;
            height: 30px;
            // margin-left: 10px;
        }

        .title {
            font-size: 9px;
            color: #f38181;
            padding: 1px 3px;
            margin-top: 5px;
            border: 1px solid #f38181;
        }
    }

    .del {
        margin-top: 15px;
        cursor: pointer;

        .delText {
            font-size: 10px;
            color: #ffc7c7;
        }
    }

    & :deep(.ant-table-thead>tr>th) {
        background-color: rgba(247, 249, 251, 1);
        text-align: center;
        padding: 10px 0;
    }

    & :deep(.ant-table-tbody>tr>td) {
        text-align: center;
    }
}
</style>