# Stroop 实验程序教程

该实验程序使用 jsPsych 库来实现经典的 Stroop 实验。Stroop 实验主要用于研究颜色词与其字体颜色之间的干扰效应。

## 实验概述

实验包含以下几个部分：

1. 欢迎页面
2. 指导语
3. 实验刺激展示
4. 实验结束页面

## 实验步骤

### 1. HTML 结构

首先，我们定义了基本的 HTML 结构，并引入了 jsPsych 相关的库和样式文件。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Stroop Experiment of jsPsych</title>
    <script src="https://unpkg.com/jspsych@7.3.4"></script>
    <script src="https://unpkg.com/@jspsych/plugin-html-keyboard-response@1.1.3"></script>
    <link href="https://unpkg.com/jspsych@7.3.4/css/jspsych.css" rel="stylesheet" type="text/css" />
    <style>
        body, html {
            height: 100%;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            background-color: #f0f0f0;
        }
        #jspsych-target {
            width: 100%;
            height: 100%;
            background-color: #f0f0f0;
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
        }
        .jspsych-content {
            font-size: 24px;
            text-align: center;
        }
        .instructions {
            margin-bottom: 20px;
            font-size: 24px;
            white-space: nowrap;
        }
        .stimulus {
            font-size: 36px;
            white-space: nowrap;
        }
    </style>
</head>
<body>
    <div id="jspsych-target"></div>
    <script>
        // JavaScript 代码将在这里编写
    </script>
</body>
</html>
```

### 2. 初始化 jsPsych

我们需要初始化 jsPsych，并定义在实验结束时的行为，例如显示数据并保存为 CSV 文件。

```
const jsPsych = initJsPsych({
    display_element: 'jspsych-target',
    on_finish: function() {
        jsPsych.data.displayData(); // 显示数据
        jsPsych.data.get().localSave('csv', 'stroop_experiment_data.csv'); // 保存实验的行为数据结果为一个 CSV 文件
    }
});
```

### 3. 创建实验时间线

定义实验时间线并添加各个部分，包括欢迎页面、指导语、实验刺激和实验结束页面。

```
var timeline = [];
```

### 4. 欢迎页面

添加一个欢迎页面，告知参与者按任意键继续。

```
var welcome = {
    type: jsPsychHtmlKeyboardResponse,
    stimulus: '<p class="instructions">欢迎您参加 Stroop 实验。请按任意键继续。</p>'
};
timeline.push(welcome);
```

### 5. 指导语

指导参与者实验的具体任务和按键反应规则。

```
var instructions = {
    type: jsPsychHtmlKeyboardResponse,
    stimulus: `
        <p class="instructions">在接下来的实验任务中，你将会看到不同的颜色词用不同的颜色显示。</p>
        <p class="instructions">如用红色字体呈现的“蓝色”字词。</p>
        <p class="instructions">而您的任务则是忽略词的含义，注意词的颜色并进行反应：</p>
        <p class="instructions"><strong>红色按 "r" ，绿色按"g" ，蓝色按"b" 。</strong></p>
        <p class="instructions">如清楚上述说明，则请按任意键开始实验。</p>
    `
};
timeline.push(instructions);
```

### 6. 设置反应试次

定义实验刺激及其呈现的颜色和含义是否一致，每种条件重复 20 次。

```
var stimuli = [
    { word: '红色', color: 'red', congruency: 'congruent' },
    { word: '红色', color: 'green', congruency: 'incongruent' },
    { word: '红色', color: 'blue', congruency: 'incongruent' },
    { word: '绿色', color: 'red', congruency: 'incongruent' },
    { word: '绿色', color: 'green', congruency: 'congruent' },
    { word: '绿色', color: 'blue', congruency: 'incongruent' },
    { word: '蓝色', color: 'red', congruency: 'incongruent' },
    { word: '蓝色', color: 'green', congruency: 'incongruent' },
    { word: '蓝色', color: 'blue', congruency: 'congruent' }
];

var trials = [];
for (var i = 0; i < stimuli.length; i++) {
    for (var j = 0; j < 20; j++) {
        trials.push({
            stimulus: `<p class="stimulus" style="color:${stimuli[i].color};">${stimuli[i].word}</p>`,
            data: {
                word: stimuli[i].word,
                color: stimuli[i].color,
                congruency: stimuli[i].congruency
            }
        });
    }
}
```

### 7. 随机呈现顺序

使用 jsPsych 提供的随机化功能对试次进行随机排序。

```
trials = jsPsych.randomization.shuffle(trials);
```

### 8. 定义试次

为每个试次定义实验逻辑，包括呈现刺激、记录反应及判断正确与否。

```
var trial = {
    type: jsPsychHtmlKeyboardResponse,
    stimulus: jsPsych.timelineVariable('stimulus'),
    choices: ['r', 'g', 'b'],
    data: jsPsych.timelineVariable('data'),
    on_finish: function(data){
        var correct = false;
        if ((data.color === 'red' && data.response === 'r') ||
            (data.color === 'green' && data.response === 'g') ||
            (data.color === 'blue' && data.response === 'b')) {
            correct = true;
        }
        data.correct = correct;
    }
};
```

### 9. 测试程序

将试次添加到时间线中，并确保试次的呈现顺序是随机的。

```
var test_procedure = {
    timeline: [trial],
    timeline_variables: trials,
    randomize_order: true
};
timeline.push(test_procedure);
```

### 10. 实验结束

定义实验结束页面，显示参与者的正确率。

```
var debrief = {
    type: jsPsychHtmlKeyboardResponse,
    stimulus: function() {
        var total_trials = jsPsych.data.get().filter({trial_type: 'html-keyboard-response'}).count();
        var accuracy = Math.round(jsPsych.data.get().filter({correct: true}).count() / total_trials * 100);
        return `<p>实验结束！感谢参与！</p><p>你的正确率是 ${accuracy}%.</p><p>按任意键结束。</p>`;
    }
};
timeline.push(debrief);
```

### 11. 启动实验

最后，启动实验时间线，开始实验。

```
jsPsych.run(timeline);
```



