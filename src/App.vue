<template>
  <div class="wolf-simulator">
    <el-button type="primary" @click="addGroup">添加狼群分组</el-button>
    <el-button type="info" @click="showConfigDialog">配置伤害系数</el-button>

    <div
      v-for="(group, groupIndex) in groups"
      :key="groupIndex"
      class="wolf-group"
    >
      <h3>狼群分组 {{ groupIndex + 1 }}</h3>
      <el-button type="success" @click="addWolf(groupIndex)" size="small"
        >添加狼</el-button
      >

      <div
        v-for="(wolf, wolfIndex) in group.wolves"
        :key="wolfIndex"
        class="wolf-card"
      >
        <div class="wolf-header">
          <span>狼 {{ wolfIndex + 1 }}</span>
          <el-button
            type="danger"
            @click="removeWolf(groupIndex, wolfIndex)"
            size="small"
            circle
          >
            <el-icon><Delete /></el-icon>
          </el-button>
        </div>

        <div class="wolf-equipment">
          <el-select
            v-model="wolf.equipment"
            placeholder="选择装备"
            @change="calculateDamage"
          >
            <el-option
              v-for="equip in getEquipmentOptions()"
              :key="equip.value"
              :label="equip.label"
              :value="equip.value"
            ></el-option>
          </el-select>
        </div>

        <div class="wolf-talents">
          <el-select
            v-model="wolf.talents"
            multiple
            placeholder="选择天赋 (最多5个)"
            :multiple-limit="5"
            @change="validateTalents(groupIndex, wolfIndex)"
          >
            <el-option
              v-for="talent in getAvailableTalents()"
              :key="talent.value"
              :label="talent.label"
              :value="talent.value"
              :disabled="isTalentDisabled(talent.value, groupIndex, wolfIndex)"
            ></el-option>
          </el-select>
        </div>

        <div class="wolf-stats">
          <h4>伤害计算</h4>
          <div v-html="wolf.calculation"></div>
          <div><strong>总伤害:</strong> {{ wolf.totalDamage.toFixed(2) }}</div>
        </div>
      </div>
    </div>

    <div class="summary">
      <h2>总计</h2>
      <p><strong>物理伤害总和:</strong> {{ totalPhysicalDamage.toFixed(2) }}</p>
      <p><strong>撕裂伤害总和:</strong> {{ totalRendingDamage.toFixed(2) }}</p>
      <p><strong>寒霜伤害总和:</strong> {{ totalFrostDamage.toFixed(2) }}</p>
      <p><strong>炽能伤害总和:</strong> {{ totalFlameDamage.toFixed(2) }}</p>
      <p><strong>电离伤害总和:</strong> {{ totalThunderDamage.toFixed(2) }}</p>
      <p><strong>总伤害:</strong> {{ grandTotalDamage.toFixed(2) }}</p>
    </div>

    <!-- 配置对话框 -->
    <el-dialog
      v-model="configDialogVisible"
      title="伤害系数配置"
      class="config-dialog"
    >
      <el-form label-width="120px" class="config-form">
        <el-form-item label="基础攻击力">
          <el-input-number
            v-model="config.baseAttack"
            :step="100"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">+{{ config.baseAttack }}</span>
        </el-form-item>
        <el-form-item label="金属爪具加成">
          <el-input-number
            v-model="config.clawBonus"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">{{ config.clawBonus * 100 }}%</span>
        </el-form-item>
        <el-form-item label="金属牙具加成">
          <el-input-number
            v-model="config.toothBonus"
            :step="100"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">+{{ config.toothBonus }}</span>
        </el-form-item>
        <el-form-item label="凶猛加成">
          <el-input-number
            v-model="config.ferociousBonus"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">{{ config.ferociousBonus * 100 }}%</span>
        </el-form-item>
        <el-form-item label="恃强凌弱加成">
          <el-input-number
            v-model="config.bullyBonus"
            :step="1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">+{{ config.bullyBonus }}</span>
        </el-form-item>
        <el-form-item label="巨兽加成">
          <el-input-number
            v-model="config.behemothBonus"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">{{ config.behemothBonus * 100 }}%</span>
        </el-form-item>
        <el-form-item label="头狼加成">
          <el-input-number
            v-model="config.alphaBonus"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">{{ config.alphaBonus * 100 }}%</span>
        </el-form-item>
        <el-form-item label="群狼加成">
          <el-input-number
            v-model="config.packBonus"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">{{ config.packBonus * 100 }}%</span>
        </el-form-item>
        <el-form-item label="自然之怒加成">
          <el-input-number
            v-model="config.natureWrathBonus"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">{{ config.natureWrathBonus * 100 }}%</span>
        </el-form-item>
        <el-form-item label="撕裂系数">
          <el-input-number
            v-model="config.rendingFactor"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">{{ config.rendingFactor * 100 }}%</span>
        </el-form-item>
        <el-form-item label="克星加成">
          <el-input-number
            v-model="config.baneBonus"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value">{{ config.baneBonus * 100 }}%</span>
        </el-form-item>
        <el-form-item label="元素之主系数">
          <el-input-number
            v-model="config.elementLordFactor"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value"
            >{{ config.elementLordFactor * 100 }}%</span
          >
        </el-form-item>
        <el-form-item label="元素之子系数">
          <el-input-number
            v-model="config.elementChildFactor"
            :step="0.1"
            :min="0"
            controls-position="right"
          />
          <span class="config-value"
            >{{ config.elementChildFactor * 100 }}%</span
          >
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="configDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="saveConfig">保存</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import { Delete } from "@element-plus/icons-vue";
import { ref, computed, watch } from "vue";
import { ElMessage } from "element-plus";

export default {
  components: { Delete },
  setup() {
    // 基础配置
    const config = ref({
      baseAttack: 4200,
      clawBonus: 0.3,
      toothBonus: 1000,
      ferociousBonus: 0.5,
      bullyBonus: 75,
      behemothBonus: 0.3,
      alphaBonus: 0.5,
      packBonus: 0.25,
      natureWrathBonus: 0.3,
      rendingFactor: 0.4,
      baneBonus: 0.6,
      elementLordFactor: 0.5,
      elementChildFactor: 0.25,
    });

    // 装备选项
    const getEquipmentOptions = () => {
      return [
        { value: "none", label: "无装备" },
        {
          value: "claw",
          label: `金属爪具 (+${config.value.clawBonus * 100}%攻击)`,
        },
        { value: "tooth", label: `金属牙具 (+${config.value.toothBonus}攻击)` },
      ];
    };

    // 天赋选项
    const getAvailableTalents = () => {
      return [
        {
          value: "ferocious",
          label: `凶猛 (+${config.value.ferociousBonus * 100}%攻击)`,
        },
        {
          value: "behemoth",
          label: `巨兽 (+${config.value.behemothBonus * 100}%最终伤害)`,
        },
        { value: "bully", label: `恃强凌弱 (+${config.value.bullyBonus}攻击)` },
        {
          value: "rending",
          label: `撕裂 (其他狼造成${
            config.value.rendingFactor * 100
          }%额外伤害)`,
        },
        {
          value: "alpha",
          label: `头狼 (+${config.value.alphaBonus * 100}%攻击, 每群狼+${
            config.value.packBonus * 100
          }%)`,
        },
        {
          value: "pack",
          label: `群狼 (头狼存在时+${config.value.packBonus * 100}%攻击)`,
        },
        { value: "battle", label: "酣战 (无加成)" },
        { value: "strong", label: "强健 (无加成)" },
        { value: "unyielding", label: "不屈 (无加成)" },
        {
          value: "natureWrath",
          label: `自然之怒 (同组+${
            config.value.natureWrathBonus * 100
          }%最终伤害)`,
        },
        {
          value: "bane",
          label: `克星 (+${config.value.baneBonus * 100}%物理攻击)`,
        },
        {
          value: "flameLord",
          label: `烈焰之主 (+${config.value.elementLordFactor * 100}%炽能伤害)`,
        },
        {
          value: "frostLord",
          label: `霜雪之主 (+${config.value.elementLordFactor * 100}%寒霜伤害)`,
        },
        {
          value: "thunderLord",
          label: `雷霆之主 (+${config.value.elementLordFactor * 100}%电离伤害)`,
        },
        {
          value: "flameChild",
          label: `烈焰之子 (+${
            config.value.elementChildFactor * 100
          }%炽能伤害)`,
        },
        {
          value: "frostChild",
          label: `霜雪之子 (+${
            config.value.elementChildFactor * 100
          }%寒霜伤害)`,
        },
        {
          value: "thunderChild",
          label: `雷霆之子 (+${
            config.value.elementChildFactor * 100
          }%电离伤害)`,
        },
      ];
    };

    // 狼群数据
    const groups = ref([
      {
        wolves: [
          {
            equipment: "none",
            talents: [],
            calculation: "",
            totalDamage: 0,
            attackPower: 0,
            physicalDamage: 0,
            rendingDamage: 0,
            flameDamage: 0,
            frostDamage: 0,
            thunderDamage: 0,
          },
        ],
      },
    ]);

    // 配置对话框
    const configDialogVisible = ref(false);

    // 添加分组
    const addGroup = () => {
      if (groups.value.length >= 5) {
        ElMessage.warning("最多只能有5个分组");
        return;
      }
      groups.value.push({
        wolves: [
          {
            equipment: "none",
            talents: [],
            calculation: "",
            totalDamage: 0,
            attackPower: 0,
            physicalDamage: 0,
            rendingDamage: 0,
            flameDamage: 0,
            frostDamage: 0,
            thunderDamage: 0,
          },
        ],
      });
      calculateDamage(); // 添加分组后重新计算
    };

    // 添加狼
    const addWolf = (groupIndex) => {
      if (groups.value[groupIndex].wolves.length >= 3) {
        ElMessage.warning("每组最多只能有3只狼");
        return;
      }
      groups.value[groupIndex].wolves.push({
        equipment: "none",
        talents: [],
        calculation: "",
        totalDamage: 0,
        attackPower: 0,
        physicalDamage: 0,
        rendingDamage: 0,
        flameDamage: 0,
        frostDamage: 0,
        thunderDamage: 0,
      });
      calculateDamage();
    };

    // 移除狼
    const removeWolf = (groupIndex, wolfIndex) => {
      // 至少保留一个分组和一只狼
      if (groups.value.length === 1 && groups.value[0].wolves.length === 1) {
        ElMessage.warning("至少需要保留一只狼");
        return;
      }

      // 移除狼
      groups.value[groupIndex].wolves.splice(wolfIndex, 1);

      // 如果分组中没有狼了，且不是最后一个分组，则移除该分组
      if (
        groups.value[groupIndex].wolves.length === 0 &&
        groups.value.length > 1
      ) {
        groups.value.splice(groupIndex, 1);
      }

      calculateDamage();
    };

    // 显示配置对话框
    const showConfigDialog = () => {
      configDialogVisible.value = true;
    };

    // 保存配置
    const saveConfig = () => {
      configDialogVisible.value = false;
      calculateDamage();
    };

    // 验证天赋选择
    const validateTalents = (groupIndex, wolfIndex) => {
      const wolf = groups.value[groupIndex].wolves[wolfIndex];

      // 检查头狼和群狼互斥
      if (wolf.talents.includes("alpha") && wolf.talents.includes("pack")) {
        wolf.talents = wolf.talents.filter((t) => t !== "pack");
        ElMessage.warning("头狼和群狼不能同时选择");
      }

      // 检查元素天赋限制
      const elementTalents = [
        "flameLord",
        "frostLord",
        "thunderLord",
        "flameChild",
        "frostChild",
        "thunderChild",
        "bane",
      ];
      const selectedElements = wolf.talents.filter((t) =>
        elementTalents.includes(t)
      );

      if (selectedElements.length > 2) {
        wolf.talents = wolf.talents.filter(
          (t) => !elementTalents.includes(t) || selectedElements.indexOf(t) < 2
        );
        ElMessage.warning("克星和元素天赋最多只能选择2个");
      }

      // 检查5个天赋时必须包含一个元素天赋
      if (wolf.talents.length === 5) {
        const hasElementTalent = wolf.talents.some((t) =>
          [
            "bane",
            "flameLord",
            "frostLord",
            "thunderLord",
            "flameChild",
            "frostChild",
            "thunderChild",
          ].includes(t)
        );
        if (!hasElementTalent) {
          wolf.talents.pop();
          ElMessage.warning("选择5个天赋时必须包含克星或元素天赋之一");
        }
      }

      calculateDamage();
    };

    // 检查天赋是否禁用
    const isTalentDisabled = (talentValue, groupIndex, wolfIndex) => {
      const wolf = groups.value[groupIndex].wolves[wolfIndex];

      // 已选中的不禁用
      if (wolf.talents.includes(talentValue)) return false;

      // 头狼和群狼互斥
      if (
        (talentValue === "alpha" && wolf.talents.includes("pack")) ||
        (talentValue === "pack" && wolf.talents.includes("alpha"))
      ) {
        return true;
      }

      // 元素天赋限制
      const elementTalents = [
        "flameLord",
        "frostLord",
        "thunderLord",
        "flameChild",
        "frostChild",
        "thunderChild",
        "bane",
      ];
      if (elementTalents.includes(talentValue) && wolf.talents.length >= 5) {
        const selectedElements = wolf.talents.filter((t) =>
          elementTalents.includes(t)
        );
        if (selectedElements.length >= 2) return true;
      }

      return false;
    };

    // 精度处理函数
    const precise = (num, decimals = 2) => {
      return parseFloat(num.toFixed(decimals));
    };

    // 在计算中所有需要显示的地方都使用这个函数

    // 计算伤害
    const calculateDamage = () => {
      // 1. 计算基础攻击力
      calculateAttackPower();

      // 2. 计算物理伤害（此时已包含所有加成）
      calculatePhysicalDamage();

      // 3. 计算撕裂伤害（必须放在物理伤害计算之后）
      calculateRendingDamage();

      // 4. 计算元素伤害
      calculateElementalAndTotalDamage();

      // 5. 计算总伤害
      calculateTotalDamage();
    };

    // 计算攻击力
    const calculateAttackPower = () => {
      // 首先计算所有狼的头狼和群狼数量
      const allWolves = groups.value.flatMap((g) => g.wolves);
      const alphaCount = allWolves.filter((w) =>
        w.talents.includes("alpha")
      ).length;
      const packCount = allWolves.filter((w) =>
        w.talents.includes("pack")
      ).length;

      // 计算每只狼的攻击力
      allWolves.forEach((wolf) => {
        let calculation = "<strong>攻击力计算:</strong><br>";

        // 基础攻击力（使用可配置值）
        let attackPower = config.value.baseAttack;
        calculation += `基础攻击力: ${config.value.baseAttack}<br>`;

        // 装备加成
        if (wolf.equipment === "tooth") {
          attackPower += config.value.toothBonus;
          calculation += `+ 金属牙具: ${config.value.toothBonus} = ${attackPower}<br>`;
        }

        // 恃强凌弱
        if (wolf.talents.includes("bully")) {
          attackPower += config.value.bullyBonus;
          calculation += `+ 恃强凌弱: ${config.value.bullyBonus} = ${attackPower}<br>`;
        }

        // 乘法部分
        let multiplier = 1;
        calculation += "<br>乘法部分:<br>初始: 1<br>";

        // 金属爪具
        if (wolf.equipment === "claw") {
          multiplier += config.value.clawBonus;
          calculation += `+ 金属爪具: ${config.value.clawBonus} = ${multiplier}<br>`;
        }

        // 凶猛
        if (wolf.talents.includes("ferocious")) {
          multiplier += config.value.ferociousBonus;
          calculation += `+ 凶猛: ${config.value.ferociousBonus} = ${multiplier}<br>`;
        }

        // 头狼
        if (wolf.talents.includes("alpha")) {
          multiplier +=
            config.value.alphaBonus + packCount * config.value.packBonus;
          calculation += `+ 头狼: ${config.value.alphaBonus} + 群狼数量(${packCount}) × ${config.value.packBonus} = ${multiplier}<br>`;
        }

        // 群狼
        if (wolf.talents.includes("pack") && alphaCount > 0) {
          multiplier += config.value.packBonus;
          calculation += `+ 群狼: ${config.value.packBonus} = ${multiplier}<br>`;
        }

        // 最终攻击力
        wolf.attackPower = attackPower * multiplier;
        calculation += `<br>最终攻击力: ${attackPower} × ${multiplier} = ${wolf.attackPower.toFixed(
          2
        )}<br>`;
        wolf.calculation = calculation;
      });
    };

    const calculatePhysicalDamage = () => {
      const allWolves = groups.value.flatMap((g) => g.wolves);

      allWolves.forEach((wolf) => {
        let calculation = wolf.calculation;
        calculation += "<br><strong>物理伤害计算:</strong><br>";

        let physicalMultiplier = 1;
        calculation += "加成系数:<br>初始: 1<br>";

        // 巨兽加成
        if (wolf.talents.includes("behemoth")) {
          physicalMultiplier = precise(
            physicalMultiplier + config.value.behemothBonus
          );
          calculation += `+ 巨兽: ${precise(
            config.value.behemothBonus
          )} = ${precise(physicalMultiplier)}<br>`;
        }

        // 自然之怒加成
        const group = groups.value.find((g) => g.wolves.includes(wolf));
        const groupWolves = group.wolves.filter((w) => w !== wolf);
        if (groupWolves.some((w) => w.talents.includes("natureWrath"))) {
          physicalMultiplier = precise(
            physicalMultiplier + config.value.natureWrathBonus
          );
          calculation += `+ 自然之怒: ${precise(
            config.value.natureWrathBonus
          )} = ${precise(physicalMultiplier)}<br>`;
        }

        // 克星加成
        if (wolf.talents.includes("bane")) {
          physicalMultiplier = precise(
            physicalMultiplier * (1 + config.value.baneBonus)
          );
          calculation += `× 克星: ${precise(
            1 + config.value.baneBonus
          )} = ${precise(physicalMultiplier)}<br>`;
        }

        // 最终物理伤害
        wolf.physicalDamage = precise(wolf.attackPower * physicalMultiplier);
        calculation += `<br>物理伤害: ${precise(wolf.attackPower)} × ${precise(
          physicalMultiplier
        )} = ${precise(wolf.physicalDamage)}<br>`;
        wolf.calculation = calculation;
      });
    };

    const calculateRendingDamage = () => {
      const allWolves = groups.value.flatMap((g) => g.wolves);

      allWolves.forEach((wolf) => {
        // 找出"排除当前狼"后的所有撕裂天赋狼
        const otherRendingWolves = allWolves.filter(
          (w) => w !== wolf && w.talents.includes("rending")
        );

        // 找出其中物理伤害最高的
        const highestRendingWolf =
          otherRendingWolves.length > 0
            ? otherRendingWolves.reduce((max, current) =>
                current.physicalDamage > max.physicalDamage ? current : max
              )
            : null;

        // 计算当前狼的撕裂伤害
        if (highestRendingWolf) {
          wolf.rendingDamage = precise(
            highestRendingWolf.physicalDamage * config.value.rendingFactor
          );
          wolf.calculation += `<br>撕裂伤害: ${precise(
            highestRendingWolf.physicalDamage
          )} (最高其他提供者) × ${precise(
            config.value.rendingFactor
          )} = ${precise(wolf.rendingDamage)}<br>`;
        } else {
          wolf.rendingDamage = 0;
          wolf.calculation += `<br>撕裂伤害: 0 (无其他撕裂提供者)<br>`;
        }
      });
    };

    const calculateElementalAndTotalDamage = () => {
      const allWolves = groups.value.flatMap((g) => g.wolves);

      allWolves.forEach((wolf) => {
        let calculation = wolf.calculation;
        calculation += "<br><strong>元素伤害计算:</strong><br>";

        // 1. 烈焰伤害计算
        let flameDamage = 0;
        if (wolf.talents.includes("flameLord")) {
          flameDamage += precise(
            wolf.physicalDamage * config.value.elementLordFactor
          );
          calculation += `烈焰之主: ${precise(wolf.physicalDamage)} × ${precise(
            config.value.elementLordFactor
          )} = ${flameDamage}<br>`;
        }
        if (wolf.talents.includes("flameChild")) {
          const childDamage = precise(
            wolf.physicalDamage * config.value.elementChildFactor
          );
          flameDamage = precise(flameDamage + childDamage);
          calculation += `烈焰之子: ${precise(wolf.physicalDamage)} × ${precise(
            config.value.elementChildFactor
          )} = ${childDamage}<br>`;
        }
        if (flameDamage > 0) {
          calculation += `→ 总炽能伤害: ${flameDamage}<br>`;
        }

        // 2. 寒霜伤害计算
        let frostDamage = 0;
        if (wolf.talents.includes("frostLord")) {
          frostDamage += precise(
            wolf.physicalDamage * config.value.elementLordFactor
          );
          calculation += `<br>霜雪之主: ${precise(
            wolf.physicalDamage
          )} × ${precise(config.value.elementLordFactor)} = ${frostDamage}<br>`;
        }
        if (wolf.talents.includes("frostChild")) {
          const childDamage = precise(
            wolf.physicalDamage * config.value.elementChildFactor
          );
          frostDamage = precise(frostDamage + childDamage);
          calculation += `霜雪之子: ${precise(wolf.physicalDamage)} × ${precise(
            config.value.elementChildFactor
          )} = ${childDamage}<br>`;
        }
        if (frostDamage > 0) {
          calculation += `→ 总寒霜伤害: ${frostDamage}<br>`;
        }

        // 3. 电离伤害计算
        let thunderDamage = 0;
        if (wolf.talents.includes("thunderLord")) {
          thunderDamage += precise(
            wolf.physicalDamage * config.value.elementLordFactor
          );
          calculation += `<br>雷霆之主: ${precise(
            wolf.physicalDamage
          )} × ${precise(
            config.value.elementLordFactor
          )} = ${thunderDamage}<br>`;
        }
        if (wolf.talents.includes("thunderChild")) {
          const childDamage = precise(
            wolf.physicalDamage * config.value.elementChildFactor
          );
          thunderDamage = precise(thunderDamage + childDamage);
          calculation += `雷霆之子: ${precise(wolf.physicalDamage)} × ${precise(
            config.value.elementChildFactor
          )} = ${childDamage}<br>`;
        }
        if (thunderDamage > 0) {
          calculation += `→ 总电离伤害: ${thunderDamage}<br>`;
        }

        // 保存计算结果
        wolf.flameDamage = flameDamage;
        wolf.frostDamage = frostDamage;
        wolf.thunderDamage = thunderDamage;

        // 4. 计算总伤害
        wolf.totalDamage = precise(
          wolf.physicalDamage +
            wolf.rendingDamage +
            flameDamage +
            frostDamage +
            thunderDamage
        );

        calculation += `<br><strong>总伤害: 
      ${precise(wolf.physicalDamage)} (物理) + 
      ${precise(wolf.rendingDamage)} (撕裂) + 
      ${flameDamage} (炽能) + 
      ${frostDamage} (寒霜) + 
      ${thunderDamage} (电离) = 
      ${precise(wolf.totalDamage)}</strong>`;

        wolf.calculation = calculation;
      });
    };

    // 计算总计
    const calculateTotalDamage = () => {
      const allWolves = groups.value.flatMap((g) => g.wolves);

      totalPhysicalDamage.value = allWolves.reduce(
        (sum, wolf) => sum + wolf.physicalDamage,
        0
      );
      totalRendingDamage.value = allWolves.reduce(
        (sum, wolf) => sum + wolf.rendingDamage,
        0
      );
      totalFlameDamage.value = allWolves.reduce(
        (sum, wolf) => sum + wolf.flameDamage,
        0
      );
      totalFrostDamage.value = allWolves.reduce(
        (sum, wolf) => sum + wolf.frostDamage,
        0
      );
      totalThunderDamage.value = allWolves.reduce(
        (sum, wolf) => sum + wolf.thunderDamage,
        0
      );
      grandTotalDamage.value = allWolves.reduce(
        (sum, wolf) => sum + wolf.totalDamage,
        0
      );
    };
    // 总计数据
    const totalPhysicalDamage = ref(0);
    const totalRendingDamage = ref(0);
    const totalFlameDamage = ref(0);
    const totalFrostDamage = ref(0);
    const totalThunderDamage = ref(0);
    const grandTotalDamage = ref(0);

    // 初始化计算
    calculateDamage();

    return {
      groups,
      getAvailableTalents,
      config,
      configDialogVisible,
      totalPhysicalDamage,
      totalRendingDamage,
      totalFlameDamage,
      totalFrostDamage,
      totalThunderDamage,
      grandTotalDamage,
      addGroup,
      addWolf,
      removeWolf,
      showConfigDialog,
      saveConfig,
      validateTalents,
      isTalentDisabled,
      calculateDamage,
      getEquipmentOptions,
    };
  },
};
</script>

<style scoped>
.wolf-simulator {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.wolf-group {
  margin: 20px 0;
  padding: 15px;
  border: 1px solid #ebeef5;
  border-radius: 4px;
  background-color: #f5f7fa;
}

.wolf-card {
  margin: 10px 0;
  padding: 15px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  background-color: white;
}

.wolf-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.wolf-equipment,
.wolf-talents {
  margin-bottom: 10px;
}

.wolf-stats {
  margin-top: 10px;
  padding: 10px;
  background-color: #f9f9f9;
  border-radius: 4px;
  font-size: 14px;
}

.summary {
  margin-top: 30px;
  padding: 20px;
  background-color: #f0f9eb;
  border-radius: 4px;
  border: 1px solid #e1f3d8;
}

h3 {
  color: #409eff;
  margin-bottom: 15px;
}

h4 {
  color: #67c23a;
  margin-bottom: 10px;
}

.config-dialog {
  max-width: 95%;
}

.config-form {
  max-height: 60vh;
  overflow-y: auto;
  padding-right: 10px;
}

.config-form .el-form-item {
  margin-bottom: 12px;
}

.config-form .el-input-number {
  width: 120px;
}

.config-value {
  margin-left: 10px;
  color: #666;
  font-size: 14px;
}

/* 响应式调整 */
@media screen and (max-width: 768px) {
  .config-dialog {
    width: 95% !important;
    max-width: 100%;
  }

  .config-form .el-form-item {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
  }

  .config-form .el-form-item__label {
    text-align: left;
    margin-bottom: 5px;
    width: 100% !important;
  }

  .config-form .el-form-item__content {
    width: 100%;
    display: flex;
    align-items: center;
  }

  .config-form .el-input-number {
    width: 100%;
    max-width: 150px;
  }

  .config-value {
    margin-left: 15px;
  }
}

@media screen and (max-width: 480px) {
  .config-form .el-form-item__content {
    flex-direction: column;
    align-items: flex-start;
  }

  .config-value {
    margin-left: 0;
    margin-top: 5px;
  }
}
</style>