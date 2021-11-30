<template>
  <div class="wrapper">
    <table id="infotable">
      <thead>
        <tr>
          <th colspan="3" style="width:27%">评价指标</th>
          <th style="width:27%">评价内容</th>
          <th style="width:6%">分值</th>
          <th v-if="bizDisplay">业务评分</th>
          <th v-if="bizDisplay">业务备注</th>
          <th v-if="riskDisplay">风控评分</th>
          <th v-if="riskDisplay">风控备注</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in dataList" :key="index">
          <td class="bolid">{{ item.index_name1 }}</td>
          <td class="bolid">{{ item.index_name2 }}</td>
          <td class="bolid">{{ item.index_name3 }}</td>
          <td>{{ item.index_name }}</td>
          <td style="text-align: center">{{ item.score }}</td>
          <template v-if="item.index_name1 == '附加'">
            <td v-if="bizDisplay">
              <h-select
                v-model="businessScore[index]"
                :clearable="false"
                algin="center"
                :disabled="editMode == 'review'"
              >
                <h-option
                  v-for="item in voteArr"
                  :value="item.value"
                  :key="item.value"
                  >{{ item.label }}</h-option
                >
              </h-select>
            </td>
            <td v-if="bizDisplay">
              <h-input
                type="text"
                v-model="businessRemark[index]"
                :disabled="editMode == 'review'"
              ></h-input>
            </td>
            <td v-if="riskDisplay">
              <h-select
                v-model="reviewScore[index]"
                :clearable="false"
                algin="center"
                :disabled="editMode == 'review'"
              >
                <h-option
                  v-for="item in voteArr"
                  :value="item.value"
                  :key="item.value"
                  >{{ item.label }}</h-option
                >
              </h-select>
            </td>
            <td v-if="riskDisplay">
              <h-input
                type="text"
                v-model="reviewRemark[index]"
                :disabled="editMode == 'review'"
              ></h-input>
            </td>
          </template>
          <template v-else>
            <td v-if="bizDisplay">
              <h-input-number
                :max="Number(item.score)"
                :min="0"
                v-model="businessScore[index]"
                size="small"
                :key="item.sort"
                focusAllSelect
                :disabled="editMode == 'review'"
              ></h-input-number>
            </td>
            <td v-if="bizDisplay">
              <h-input
                type="text"
                :maxlength="500"
                v-model="businessRemark[index]"
                :disabled="editMode == 'review'"
              ></h-input>
            </td>
            <td v-if="riskDisplay">
              <h-input-number
                :max="Number(item.score)"
                :min="0"
                focusAllSelect
                v-model="reviewScore[index]"
                size="small"
                :key="item.sort"
                :disabled="editMode == 'review'"
              ></h-input-number>
            </td>
            <td v-if="riskDisplay">
              <h-input
                type="text"
                :maxlength="500"
                v-model="reviewRemark[index]"
                :disabled="editMode == 'review'"
              ></h-input>
            </td>
          </template>
        </tr>
      </tbody>
      <tfoot style="text-align: center;font-weight: bold;">
        <tr>
          <td colspan="4">合计得分：</td>
          <td>100</td>
          <td v-if="bizDisplay">
            <h-input-number
              disabled
              v-model="businessScoreAll"
              size="small"
            ></h-input-number>
          </td>
          <td v-if="bizDisplay"></td>
          <td v-if="riskDisplay">
            <h-input-number
              disabled
              v-model="reviewScoreAll"
              size="small"
            ></h-input-number>
          </td>
          <td v-if="riskDisplay"></td>
        </tr>
        <tr v-if="riskDisplay && bizDisplay">
          <td colspan="4">
            加权评价：（业务评分*70%+风控评分*30%）
          </td>
          <td></td>
          <td colspan="4">
            <h-input-number
              disabled
              v-model="total_score"
              precision="2"
              size="small"
            ></h-input-number>
          </td>
        </tr>
      </tfoot>
    </table>
  </div>
</template>
<script>
import clientform from "@trust-base-common/mixins/wf/tbaseclientform";
import {
  accAdd,
  accSub,
  accMul,
  toPoint,
} from "@amc-business-web/utils/common/calculator.js";

export default {
  data() {
    return {
      bizDisplay: null, //业务
      riskDisplay: null, //风控

      businessScore: [], //业务评分
      reviewScore: [], //风评评分
      businessRemark: [], //业务备注
      reviewRemark: [], //风评备注
      detail_id: [], //评价ID

      businessScoreAll: 0, //业务评分总和
      reviewScoreAll: 0, //风评评分总和
      total_score: 0.0, //加权评价
      voteArr: [
        { label: "是", value: 1 },
        { label: "否", value: 0 },
      ],
      dataList: [
        {
          index_code: "agency_01010101",
          index_name: "1.是否按时完成各项工作",
          parent_index_code: "agency_010101",
          index_type: "agency_eval",
          score: "3",
          sort: 1,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "工作能力(20分)",
          index_name3: "工作效率(6分)",
        },
        {
          index_code: "agency_01010102",
          index_name: "2.能否按照计划进度开展工作",
          parent_index_code: "agency_010101",
          index_type: "agency_eval",
          score: "3",
          sort: 2,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "工作能力(20分)",
          index_name3: "工作效率(6分)",
        },
        {
          index_code: "agency_01010201",
          index_name: "3.现场工作期间，工作人数，人员资质等是否达到委托要求",
          parent_index_code: "agency_010102",
          index_type: "agency_eval",
          score: "4",
          sort: 3,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "工作能力(20分)",
          index_name3: "胜任能力(14分)",
        },
        {
          index_code: "agency_01010202",
          index_name: "4.工作人员是否具备与服务项目相匹配的专业知识",
          parent_index_code: "agency_010102",
          index_type: "agency_eval",
          score: "4",
          sort: 4,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "工作能力(20分)",
          index_name3: "胜任能力(14分)",
        },
        {
          index_code: "agency_01010203",
          index_name:
            "5.是否存在对同一事项（如差错，修改意见，疑问等）进行多次修改才得到最终结果的情况",
          parent_index_code: "agency_010102",
          index_type: "agency_eval",
          score: "6",
          sort: 5,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "工作能力(20分)",
          index_name3: "胜任能力(14分)",
        },
        {
          index_code: "agency_01020101",
          index_name: "6.是否及时就项目进度，情况进行沟通",
          parent_index_code: "agency_010201",
          index_type: "agency_eval",
          score: "2",
          sort: 6,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "配合程度(10分)",
          index_name3: "沟通主动性(6分)",
        },
        {
          index_code: "agency_01020102",
          index_name:
            "7.是否及时回馈委托方提出的意见问题等，并主动提出有关建议等",
          parent_index_code: "agency_010201",
          index_type: "agency_eval",
          score: "2",
          sort: 7,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "配合程度(10分)",
          index_name3: "沟通主动性(6分)",
        },
        {
          index_code: "agency_01020103",
          index_name: "8.对重大事项得出结论前是否与有关方面进行了充分的沟通",
          parent_index_code: "agency_010201",
          index_type: "agency_eval",
          score: "2",
          sort: 8,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "配合程度(10分)",
          index_name3: "沟通主动性(6分)",
        },
        {
          index_code: "agency_01020201",
          index_name: "9.工作人员能否准确理解委托方的意图",
          parent_index_code: "agency_010202",
          index_type: "agency_eval",
          score: "2",
          sort: 9,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "配合程度(10分)",
          index_name3: "沟通能力(4分)",
        },
        {
          index_code: "agency_01020202",
          index_name: "10.工作人员表达或表述，是否清晰无误，善于沟通",
          parent_index_code: "agency_010202",
          index_type: "agency_eval",
          score: "2",
          sort: 10,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "配合程度(10分)",
          index_name3: "沟通能力(4分)",
        },
        {
          index_code: "agency_01030101",
          index_name:
            "11.工作团队对事项的描述，问题的定位和数据的确定等是否真实谨慎",
          parent_index_code: "agency_010301",
          index_type: "agency_eval",
          score: "2",
          sort: 11,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "职业操守(10分)",
          index_name3: "工作态度(4分)",
        },
        {
          index_code: "agency_01030102",
          index_name:
            "12.工作团队对事项的描述，问题的定位和数据的确定等是否客观真实",
          parent_index_code: "agency_010301",
          index_type: "agency_eval",
          score: "2",
          sort: 12,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "职业操守(10分)",
          index_name3: "工作态度(4分)",
        },
        {
          index_code: "agency_01030201",
          index_name: "13.工作人员能否对执业过程中获得的信息保密",
          parent_index_code: "agency_010302",
          index_type: "agency_eval",
          score: "3",
          sort: 13,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "职业操守(10分)",
          index_name3: "职业道德(6分)",
        },
        {
          index_code: "agency_01030202",
          index_name: "14.工作人员能否坚持独立，客观，公正的立场",
          parent_index_code: "agency_010302",
          index_type: "agency_eval",
          score: "3",
          sort: 14,
          level: 4,
          version: "1",
          index_name1: "服务过程(40分)",
          index_name2: "职业操守(10分)",
          index_name3: "职业道德(6分)",
        },
        {
          index_code: "agency_02010101",
          index_name: "15.报告和相关资料的格式是否符合委托方的要求/是否规范",
          parent_index_code: "agency_020101",
          index_type: "agency_eval",
          score: "10",
          sort: 15,
          level: 4,
          version: "1",
          index_name1: "服务成果(60分)",
          index_name2: "规范性(30分)",
          index_name3: "格式与文字(30分)",
        },
        {
          index_code: "agency_02010102",
          index_name: "16.报告和相关材料的数据是否正确",
          parent_index_code: "agency_020101",
          index_type: "agency_eval",
          score: "10",
          sort: 16,
          level: 4,
          version: "1",
          index_name1: "服务成果(60分)",
          index_name2: "规范性(30分)",
          index_name3: "格式与文字(30分)",
        },
        {
          index_code: "agency_02010103",
          index_name: "17.报告相关依据是否合理，充分",
          parent_index_code: "agency_020101",
          index_type: "agency_eval",
          score: "10",
          sort: 17,
          level: 4,
          version: "1",
          index_name1: "服务成果(60分)",
          index_name2: "规范性(30分)",
          index_name3: "格式与文字(30分)",
        },
        {
          index_code: "agency_02020101",
          index_name: "18.是否完成协议规定的具体服务内容",
          parent_index_code: "agency_020201",
          index_type: "agency_eval",
          score: "15",
          sort: 18,
          level: 4,
          version: "1",
          index_name1: "服务成果(60分)",
          index_name2: "完整性(30分)",
          index_name3: "内容与成果(30分)",
        },
        {
          index_code: "agency_02020102",
          index_name: "19.是否按要求提交相关的工作成果文件资料",
          parent_index_code: "agency_020201",
          index_type: "agency_eval",
          score: "15",
          sort: 19,
          level: 4,
          version: "1",
          index_name1: "服务成果(60分)",
          index_name2: "完整性(30分)",
          index_name3: "内容与成果(30分)",
        },
        {
          index_code: "agency_03010101",
          index_name: "20.中介机构工作人员是否存在违法违规及公司明令禁止的行为",
          parent_index_code: "agency_030101",
          index_type: "agency_eval",
          score: "是/否",
          sort: 20,
          level: 4,
          version: "1",
          index_name1: "附加",
          index_name2: "一票否决项",
          index_name3: "一票否决项",
        },
      ], //表格展示数据
      indexDetails: [], //表格参数
    };
  },
  props: {
    node_variable: {
      type: [Object, Array],
      default: [],
    },
    editMode: {
      type: Boolean,
      default: false,
    },
  },
  mixins: [clientform],
  watch: {
    // 监听业务评分整列
    businessScore(newVal, oldVal) {
      // 判断是否一票否决
      if (newVal[newVal.length - 1] == 1) {
        this.businessScoreAll = 0;
      } else {
        this.businessScoreAll = newVal.reduce((x, y) => Number(x) + Number(y));
      }
      this.total_score = accAdd(
        accMul(this.businessScoreAll, 0.7),
        accMul(this.reviewScoreAll, 0.3)
      );

      // this.businessScoreAll * 0.7 + this.reviewScoreAll * 0.3;
    },
    // 监听风控评分整列
    reviewScore(newVal, oldVal) {
      if (newVal[newVal.length - 1] == 1) {
        this.reviewScoreAll = 0;
      } else {
        this.reviewScoreAll = newVal.reduce((x, y) => Number(x) + Number(y));
      }
      this.total_score = accAdd(
        accMul(this.businessScoreAll, 0.7),
        accMul(this.reviewScoreAll, 0.3)
      );
      // this.businessScoreAll * 0.7 + this.reviewScoreAll * 0.3;
    },
  },
  mounted() {
    this.init();
    //控制业务、风控列的显隐  值为 0 不显示   1 显示
    this.bizDisplay = Number(this.node_variable.bizDisplay); //业务
    this.riskDisplay = Number(this.node_variable.riskDisplay); //风控
  },
  methods: {
    // 获取表格可编辑的内容数据
    change() {
      this.indexDetails = [];
      for (let i = 0; i < this.dataList.length; i++) {
        this.indexDetails.push({
          indexCode: this.dataList[i].index_code,
          businessScore: this.businessScore[i],
          riskScore: this.reviewScore[i],
          businessDesc: this.businessRemark[i],
          riskDesc: this.reviewRemark[i],
          detail_id: this.detail_id[i],
        });
      }
      return this.indexDetails;
    },
    /**
     * [mergeSameCell 单元格合并]
     * @param  {[type]} dom      [table对应的dom元素]
     * @param  {[type]} beginRow [从第几行开始合并（从0开始）]
     * @param  {[type]} endRow   [合并到哪一行，负数表示从底下数几行不合并]
     * @param  {[type]} colIdxes [合并的列下标的数组，如[0,1]表示合并前两列，[0]表示只合并第一列]
     * @return {[type]}          [description]
     */
    mergeSameCell(dom, beginRow, endRow, colIdxes) {
      var colIdx = colIdxes[0];
      var newColIdxes = colIdxes.concat();
      newColIdxes.splice(0, 1);
      var delRows = new Array();
      var rs = dom.rows;
      //endRow为0的时候合并到最后一行，小于0时表示最后有-endRow行不合并
      if (endRow === 0) {
        endRow = rs.length - 1;
      } else if (endRow < 0) {
        endRow = rs.length - 1 + endRow;
      }
      var rowSpan = 1; // 要设置的rowSpan的值
      var rowIdx = beginRow; // 要设置rowSpan的cell行下标
      var cellValue; // 存储单元格里面的内容
      for (var i = beginRow; i <= endRow + 1; i++) {
        if (i === endRow + 1) {
          //过了最后一行的时候合并前面的单元格
          if (newColIdxes.length > 0) {
            this.mergeSameCell(dom, rowIdx, endRow, newColIdxes);
          }
          rs[rowIdx].cells[colIdx].rowSpan = rowSpan;
        } else {
          var cell = rs[i].cells[colIdx];
          if (i === beginRow) {
            // 第一行的时候初始化各个参数
            cellValue = cell.innerHTML;
            rowSpan = 1;
            rowIdx = i;
          } else if (cellValue != cell.innerHTML) {
            // 数据改变合并前面的单元格
            cellValue = cell.innerHTML;
            if (newColIdxes.length > 0) {
              this.mergeSameCell(dom, rowIdx, i - 1, newColIdxes);
            }
            rs[rowIdx].cells[colIdx].rowSpan = rowSpan;
            rowSpan = 1;
            rowIdx = i;
          } else if (cellValue === cell.innerHTML) {
            // 数据和前面的数据重复的时候删除单元格
            rowSpan++;
            delRows.push(i);
          }
        }
      }
      for (var j = 0; j < delRows.length; j++) {
        rs[delRows[j]].deleteCell(colIdx);
      }
    },

    //获取表格展示数据
    init() {
      // let params = { index_type: "agency_eval" };
      // let url = TBaseHttp.appServer(
      //   window.LOCAL_CONFIG.APP_SERVER_NAME_MAP.AMCBUSINESS,
      //   "/intermediaryAgencyEvalService/getEvalIndexConfig"
      // );
      // this.$TBaseHttp.postComplexJson(url, params, (response) => {
      //   if (response && response.data && response.data.status == 1) {
      //     this.dataList = response.data.data_list;
          this.initContent(); //初始化表格待填信息
          this.$nextTick(function() {
            this.mergeSameCell(document.getElementById("infotable"), 1, 0, [
              0,
              1,
              2,
            ]);
          });
      //   } else {
      //     this.$hMessage.warning(response.data.error_info);
      //   }
      // });
    },
    // 针对表格可填数据的填值以及回显
    initContent(datas) {
      if (datas) {
        this.businessScore = datas.map((_) => Number(_.business_score));
        this.reviewScore = datas.map((_) => Number(_.risk_score));
        this.businessRemark = datas.map((_) => _.business_desc);
        this.reviewRemark = datas.map((_) => _.risk_desc);
        this.detail_id = datas.map((_) => _.detail_id);
      } else {
        this.businessScore = new Array(this.dataList.length).fill(0);
        this.reviewScore = new Array(this.dataList.length).fill(0);
        this.businessRemark = new Array(this.dataList.length).fill("");
        this.reviewRemark = new Array(this.dataList.length).fill("");
        this.detail_id = new Array(this.dataList.length).fill("");
      }
    },
  },
};
</script>
<style scoped>
.bolid {
  color: black;
  font-weight: bold;
  text-align: center;
}
table {
  width: 100%;
  border: 1px solid #ccc;
  border-collapse: collapse;
}
thead th {
  text-align: center;
  background-color: #fafafa;
}
th,
td {
  padding: 3px 5px;
  line-height: 25px;
  border: 1px solid #e9eaec;
}
.wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
