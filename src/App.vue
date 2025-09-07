<template>
  <div class="mx-auto max-w-4xl px-4 pt-20 py-8 bg-gray-100">
    <h1 class="text-2xl font-bold mb-4">GitHub Contribution Calendar</h1>
    <ActiveGraph
      :contributions="convertCommitsToContributionData(demoData)"
      :total="total"
      lang="zh"
      @click="handleDayClick" />
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import ActiveGraph from "./components/ActiveGraph.vue";
  import dayjs from "dayjs";
  import isSameOrBefore from "dayjs/plugin/isSameOrBefore";
  dayjs.extend(isSameOrBefore);
  import demoData from "./assets/changelog.json";

  const total = ref(0);
  // Generate random contribution data for the past year
  const generateContributionData = () => {
    const startDate = new Date(
      new Date().setFullYear(new Date().getFullYear() - 1)
    );
    const endDate = new Date();
    const contributionData = [];
    let totalCount = 0;

    const formatDate = (date) => {
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, "0");
      const day = String(date.getDate()).padStart(2, "0");
      return `${year}-${month}-${day}`;
    };

    let currentDate = startDate;

    while (currentDate <= endDate) {
      const weekData = [];

      for (let i = 0; i < 7; i++) {
        if (currentDate > endDate) break;

        const count = Math.floor(Math.random() * 11);
        totalCount += count;

        weekData.push({
          date: formatDate(currentDate),
          intensity: Math.floor(Math.random() * 5),
          count,
        });

        currentDate.setDate(currentDate.getDate() + 1);
      }

      contributionData.push(weekData);
    }

    total.value = totalCount;
    return contributionData;
  };

  // Convert commit data to contribution data
  const convertCommitsToContributionData = (commits) => {
    if (!commits || commits.length === 0) return [];

    const commitCounts = {};
    commits.forEach((commit) => {
      const dateStr = dayjs(commit.date).format("YYYY-MM-DD");
      commitCounts[dateStr] = (commitCounts[dateStr] || 0) + 1;
    });

    // 固定过去 53 周
    const endDate = dayjs().endOf("day");
    const startDate = endDate.subtract(52, "week").startOf("week"); // 周日对齐

    const contributionData = [];
    let currentDate = startDate;
    let totalCount = 0;

    while (currentDate.isSameOrBefore(endDate, "day")) {
      const week = [];
      for (let i = 0; i < 7; i++) {
        if (currentDate.isAfter(endDate, "day")) break;

        const dateStr = currentDate.format("YYYY-MM-DD");
        const count = commitCounts[dateStr] || 0;
        totalCount += count;

        let intensity = 0;
        if (count > 0) intensity = 1;
        if (count >= 3) intensity = 2;
        if (count >= 5) intensity = 3;
        if (count >= 8) intensity = 4;

        week.push({
          date: dateStr,
          count,
          intensity,
        });

        currentDate = currentDate.add(1, "day");
      }
      contributionData.push(week);
    }

    total.value = totalCount;
    return contributionData;
  };

  const handleDayClick = (day) => {
    if (day.ignore || !day.date) return;
    const commitCount = day.count;
    const date = day.date;
    alert(`日期: ${date} 提交数: ${commitCount}`);
  };
</script>
