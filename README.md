# ConvLab-2
**ConvLab-2** is an open-source toolkit that enables researchers to build task-oriented dialogue systems with state-of-the-art models, perform an end-to-end evaluation, and diagnose the weakness of systems. As the successor of [ConvLab](https://github.com/ConvLab/ConvLab), ConvLab-2 inherits ConvLab's framework but integrates more powerful dialogue models and supports more datasets. Besides, we have developed an analysis tool and an interactive tool to assist researchers in diagnosing dialogue systems. [[paper]](https://arxiv.org/abs/2002.04793)

- [Installation](#installation)
- [Tutorials](#tutorials)
- [Models](#models)
- [Supported Dataset](#Supported-Dataset)
- [End-to-end Performance on MultiWOZ](#End-to-end-Performance-on-MultiWOZ)
- [Module Performance on MultiWOZ](#Module-Performance-on-MultiWOZ)
- [Issues](#issues)
- [Contributions](#contributions)
- [Citing](#citing)
- [License](#license)

## Installation

Require python 3.6.

Clone this repository:
```bash
git clone https://github.com/thu-coai/ConvLab-2.git
```

Install ConvLab-2 via pip:

```bash
cd ConvLab-2
pip install -e .
```

## Tutorials

- [Getting Started](https://github.com/thu-coai/ConvLab-2/blob/master/tutorials/Getting_Started.ipynb) (Have a try on [Colab](https://colab.research.google.com/github/thu-coai/ConvLab-2/blob/master/tutorials/Getting_Started.ipynb)!)
- [Add New Model](https://github.com/thu-coai/ConvLab-2/blob/master/tutorials/Add_New_Model.md)
- [Train RL Policies](https://github.com/thu-coai/ConvLab-2/blob/master/tutorials/Train_RL_Policies)
- [Interactive Tool](https://github.com/thu-coai/ConvLab-2/blob/master/deploy) [[demo video]](https://drive.google.com/file/d/1HR3mjhgLL0g9IbqU443NsH2G0-PpAsog/view?usp=sharing)

## Models

We provide following models:

- NLU: SVMNLU, MILU, BERTNLU
- DST: rule, MDBT, TRADE, SUMBT
- Policy: rule, Imitation, REINFORCE, PPO, GDPL, MDRG, HDSA, LaRL
- Simulator policy: Agenda, VHUS
- NLG: Template, SCLSTM
- End2End: Sequicity, DAMD, RNN_rollout

For  more details about these models, You can refer to `README.md` under `convlab2/$module/$model/$dataset` dir such as `convlab2/nlu/jointBERT/multiwoz/README.md`.

## Supported Datasets

- [Multiwoz 2.1](https://github.com/budzianowski/multiwoz)
  - We add user dialogue act (*inform*, *request*, *bye*, *greet*, *thank*), remove 5 sessions that have incomplete dialogue act annotation and place it under `data/multiwoz` dir.
  - Train/val/test size: 8434/999/1000. Split as original data.
  - LICENSE: Attribution 4.0 International, url: http://creativecommons.org/licenses/by/4.0/
- [CrossWOZ](https://github.com/thu-coai/CrossWOZ)
  - We offers a rule-based user simulator and a complete set of models for building a pipeline system on the CrossWOZ dataset. We correct few state annotation and place it under `data/crosswoz` dir.
  - Train/val/test size: 5012/500/500. Split as original data.
  - LICENSE: Attribution 4.0 International, url: http://creativecommons.org/licenses/by/4.0/
- [Camrest](https://www.repository.cam.ac.uk/handle/1810/260970)
  - We add system dialogue act (*inform*, *request*, *nooffer*) and place it under `data/camrest` dir.
  - Train/val/test size: 406/135/135. Split as original data.
  - LICENSE: Attribution 4.0 International, url: http://creativecommons.org/licenses/by/4.0/
- [Dealornot](https://github.com/facebookresearch/end-to-end-negotiator/tree/master/src/data/negotiate)
  - Placed under `data/dealornot` dir.
  - Train/val/test size: 5048/234/526. Split as original data.
  - LICENSE: Attribution-NonCommercial 4.0 International, url: https://creativecommons.org/licenses/by-nc/4.0/

## End-to-end Performance on MultiWOZ

We perform end-to-end evaluation (1000 dialogues) on MultiWOZ using the user simulator below (a full example on `tests/test_end2end.py`) :

```python
# BERT nlu trained on sys utterance
user_nlu = BERTNLU(mode='sys', config_file='multiwoz_sys_context.json', model_file='https://convlab.blob.core.windows.net/convlab-2/bert_multiwoz_sys_context.zip')
user_dst = None
user_policy = RulePolicy(character='usr')
user_nlg = TemplateNLG(is_user=True)
user_agent = PipelineAgent(user_nlu, user_dst, user_policy, user_nlg, name='user')

analyzer = Analyzer(user_agent=user_agent, dataset='multiwoz')

set_seed(20200202)
analyzer.comprehensive_analyze(sys_agent=sys_agent, model_name='sys_agent', total_dialog=1000)
```

Main metrics (refer to `convlab2/evaluator/multiwoz_eval.py` for more details):

- Complete: whether complete the goal. Judged by the Agenda policy instead of external evaluator.
- Success: whether all user requests have been informed and the booked entities satisfy the constraints.
- Book: how many the booked entities satisfy the user constraints.
- Inform Precision/Recall/F1: how many user requests have been informed.
- Turn(succ/all): average turn number for successful/all dialogues.

Performance (the first row is the default config for each module. Empty entries are set to default config.):

| NLU         | DST       | Policy         | NLG         | Complete rate | Success rate | Book rate | Inform P/R/F1 | Turn(succ/all) |
| ----------- | --------- | -------------- | ----------- | ------------- | ------------ | --------- | --------- | -------------- |
| **BERTNLU** | RuleDST   | RulePolicy     | TemplateNLG | 92.1          | 85.5         | 91.5      | 79.8/92.8/83.8 | 12.7/13.8      |
| **MILU**    | RuleDST | RulePolicy | TemplateNLG | 89.9          | 83.1         | 90.9      | 78.3/91.7/82.5 | 12.1/13.9      |
| **SVMNLU**  | RuleDST | RulePolicy | TemplateNLG | 84.2          | 70.4         | 76.1      | 79.1/88.8/81.5 | 14.8/17.7      |
| BERTNLU | RuleDST | RulePolicy | **SCLSTM**  | 40.1       | 41.0  | 51.5    | 68.5/56.5/59.1 |      11.6/29.2      |
| BERTNLU     | RuleDST | **MLEPolicy**  | TemplateNLG | 52.6              | 48.4         | 35.5      |  66.3/72.7/66.0 | 12.5/26.3      |
| BERTNLU | RuleDST | **PGPolicy**   | TemplateNLG | 42.9              | 43.3         | 31.0      |  61.9/66.8/60.4 | 14.7/29.1      |
| BERTNLU | RuleDST | **PPOPolicy**  | TemplateNLG | 69.7              | 56.6         | 56.6      |  64.8/79.0/68.1 | 12.9/22.1      |
| BERTNLU | RuleDST | **GDPLPolicy** | TemplateNLG | 57.9              | 49.5         | 33.5      |  67.0/76.4/68.2 | 11.5/24.3      |
| None        | **MDBT**  | RulePolicy | TemplateNLG |     27.7      |       21.2     |   45.4    |  52.2/41.0/42.4 |   11.8/32.1       |
| None        | **TRADE** | RulePolicy | TemplateNLG |      29.9      |    25.3       |     36.9     | 49.3/48.1/44.4 |     12.7/24.7     |
| None        | **SUMBT** | RulePolicy | TemplateNLG |       34.7    |    33.8     |    57.8   |  52.3/50.6/47.3   | 12.1/26.6         |
| BERTNLU | RuleDST | **MDRG**       | None        | 27.0 | 25.2 | 49.0 | 46.6/43.1/42.0 | 13.6/33.6 |
| BERTNLU | RuleDST | **HDSA**       | None        | 35.6 | 27.5 | 5.4 | 47.8/57.2/48.8 | 13.0/31.5 |
| BERTNLU | RuleDST | **LaRL**       | None        | 40.6 | 34.0 | 45.6 | 47.8/54.1/47.6 | 15.0/28.6 |
| None | **SUMBT** | **LaRL** | None |    39.4|   33.1| 39.5  | 48.5/56.0/48.8| 15.5/28.7|
| None | None | **Sequicity*** | None | 13.1 | 10.5 | 5.1 | 41.4/30.8/31.3 | 12.9/38.3 |
| None | None | **DAMD***      | None | 38.5 | 33.6 | 50.9 | 62.1/60.7/57.4 | 10.4/28.2 |

*: end-to-end models used as sys_agent directly.

## Module Performance on MultiWOZ

### NLU

By running `convlab2/nlu/evaluate.py MultiWOZ $model all`:

|         | Precision | Recall | F1    |
| ------- | --------- | ------ | ----- |
| BERTNLU | 82.48     | 85.59  | 84.01 |
| MILU    | 80.29     | 83.63  | 81.92 |
| SVMNLU  | 74.96     | 50.74  | 60.52 |

### DST 

By running `convlab2/dst/evaluate.py MultiWOZ $model`:

|             |  Joint accuracy  | Slot accuracy | Joint F1  |
| --------    |   -------------   | -------------  | --------|
|  MDBT       |   0.06           |      0.89       | 0.43    |
|  SUMBT      |    0.30         |       0.96       | 0.83    |
|   TRADE     |    0.40         |       0.96       | 0.84    |

### Policy

By running `convlab2/policy/evalutate.py --model_name $model`

|           | Task Success Rate |
| --------- | ----------------- |
| MLE       | 0.56              |
| PG        | 0.54              |
| PPO       | 0.74              |
| GDPL      | 0.58              |

### NLG

By running `convlab2/nlg/evaluate.py MultiWOZ $model sys`

|          | corpus BLEU-4 |
| -------- | ------------- |
| Template | 0.3309        |
| SCLSTM   | 0.4884        |

## Issues

You are welcome to create an issue if you want to request a feature, report a bug or ask a general question.

## Contributions

We welcome contributions from community.

- If you want to make a big change, we recommend first creating an issue with your design.
- Small contributions can be directly made by a pull request.
- If you like make contributions to our library, see issues to find what we need.

## Team

**ConvLab-2** is maintained and developed by Tsinghua University Conversational AI group (THU-coai) and Microsoft Research (MSR).

We would like to thank:

Yan Fang, Zhuoer Feng, Jianfeng Gao, Qihan Guo, Kaili Huang, Minlie Huang, Sungjin Lee, Bing Li, Jinchao Li, Xiang Li, Xiujun Li, Wenchang Ma, Baolin Peng, Runze Liang, Ryuichi Takanobu, Jiaxin Wen, Yaoqin Zhang, Zheng Zhang, Qi Zhu, Xiaoyan Zhu.


## Citing

If you use ConvLab-2 in your research, please cite:

```
@inproceedings{zhu2020convlab2,
    title={ConvLab-2: An Open-Source Toolkit for Building, Evaluating, and Diagnosing Dialogue Systems},
    author={Qi Zhu and Zheng Zhang and Yan Fang and Xiang Li and Ryuichi Takanobu and Jinchao Li and Baolin Peng and Jianfeng Gao and Xiaoyan Zhu and Minlie Huang},
    year={2020},
    booktitle={Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics},
}
```

## License

Apache License 2.0
