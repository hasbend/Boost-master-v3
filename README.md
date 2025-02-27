# AndroidAPS - Boost 
* Check the wiki: https://androidaps.readthedocs.io
*  Everyone who’s been looping with AndroidAPS needs to fill out the form after 3 days of looping  https://docs.google.com/forms/d/14KcMjlINPMJHVt28MDRupa4sz4DDIooI4SrW0P3HSN8/viewform?c=0&w=1

[![Support Server](https://img.shields.io/discord/629952586895851530.svg?label=Discord&logo=Discord&colorB=7289da&style=for-the-badge)](https://discord.gg/4fQUWHZ4Mw)

[![CircleCI](https://circleci.com/gh/nightscout/AndroidAPS/tree/master.svg?style=svg)](https://circleci.com/gh/nightscout/AndroidAPS/tree/master)
[![Crowdin](https://d322cqt584bo4o.cloudfront.net/androidaps/localized.svg)](https://translations.androidaps.org/project/androidaps)
[![Documentation Status](https://readthedocs.org/projects/androidaps/badge/?version=latest)](https://androidaps.readthedocs.io/en/latest/?badge=latest)
[![codecov](https://codecov.io/gh/nightscout/AndroidAPS/branch/master/graph/badge.svg)](https://codecov.io/gh/MilosKozak/AndroidAPS)

DEV: 
[![CircleCI](https://circleci.com/gh/nightscout/AndroidAPS/tree/dev.svg?style=svg)](https://circleci.com/gh/nightscout/AndroidAPS/tree/dev)
[![codecov](https://codecov.io/gh/nightscout/AndroidAPS/branch/dev/graph/badge.svg)](https://codecov.io/gh/MilosKozak/AndroidAPS)


<img src="https://cdn.iconscout.com/icon/free/png-256/bitcoin-384-920569.png" srcset="https://cdn.iconscout.com/icon/free/png-512/bitcoin-384-920569.png 2x" alt="Bitcoin Icon" width="100">

3KawK8aQe48478s6fxJ8Ms6VTWkwjgr9f2

**Boost 3.6.5**

Traditional Autosens is deprecated in this code and sensitivityRatio is calculated using 'Eight hour weighted average TDD  / 7-day average TDD'.

Boost uses a similar version of DynamicISF for making predictions, however, unlike the hardoded quanta for the different values of insulin peak, when free-peak is used, it scales between the highest and lowest values. It also enables variation of the Dynamic ISF adjustment factor with profile percentage changes. 

The ISF for dosing decisions within Boost is slighty different to thhat in DynamicISF. The calculation is intended to mimic the effects of higher insulin sensitivty at lower glucose levels, and runs as follows:

1. With COB and increasing deltas, use 75% of the predicted BG and 25% of the current BG.
2. If current BG is accelerating fast, bg is below 180mg/dl/10mmol/l and eventual BG is higher than current, use 50% of both eventual and current BG.
3. If BG is between 160 and 270 and almost flat, and eventual BG is lower than BG, use 60% min predicted BG and 40% current BG.
4. If BG is increasing slowly, and above 198 or eventual BG is above current BG and current BG is above 198,  use 40% min predicted BG and 60% current BG.
5. If BG is increasing more slowy, or eventual BG is greater than current BG, use current BG.
6. If BG is not increasing, use minimum predicted BG. 

In Treatments Safety in preferences, there is now a user adjustable Low Glucose Suspend threshold. This allows the user to set a value higher than the system would normally use, such that when predictions drop below this level, a zero TBR is set.

You can use Boost when announcing carbs or without announcing carbs. With COB there is an additional piece of bolusing code that oeprates for the first 40 mins of COB. If you prefer to manually bolus, it fully supports that with no other code.

It also has variable insulin percentage determined by the user, and while boost time is valid, the algorithm can bolus up to a maximum bolus defined by the user in preferences.

The intention of this code is to deliver an early, larger bolus when rises are detected to intiate UAM deviations and to allow the algorithm to be more aggressive. Other than Boost, it relies on oref1 adjusted to use the variable ISF function based on TDD.

All of the additional code outside of the standard SMB calculation requires a time period to be specified within which it is active. The default time settings disable the code. The time period is specified in hours using a 24 hour clock in the Boost preferences section.

**COB:** With Carbs on Board, Boost has a 15 minute window to deliver the equivalent of a mealtime bolus and **is allowed to go higher than your Boost Bolus Cap**, up to `InsulinRequired/insulin required percent` calculated by the oref1 algorithm, taking carbs into account. In the following 40 mins after the carbs are added, it can do additional larger boluses, as long as there is a delta >5 and COB > 0. The max allowed is the greater of the Boost Bolus Cap or the "COB cap", which is calculated as `COB / Carb Ratio`.

During normal use, you should set your Boost Bolus Cap to be the max that boost delivers when Boost is enabled and no COB are entered.

Boost outside the first 40 mins of COB, or with 0 COB has four phases:

1. Boost bolus
2. Percentage Scale
3. Enhanced oref1
4. Regular oref1

**Boost**

When an initial rise is detected with a meal, but no announced COB, delta, short_avgDelta and long_avgDelta are used to trigger the early bolus (assuming IOB is below a user defined amount). The early bolus value is one hour of basal requirement and is based on the current period basal rate, unless this is smaller than "Insulin Required" when that is used instead.

The user defined Boost Scale Value can be used to increase the boost bolus if the user requires, however, users should be aware that this increases the risk of hypos when small rises occur.

If **Boost Scale Value** is less than 3, Boost is enabled.

The short and long average delta clauses disable boost once delta and the average deltas are aligned. There is a preferences setting (Boost Bolus Cap) that limits the maximum bolus that can be delivered by Boost outside of the standard UAMSMBBasalMinutes limit.

**Boost Percentage Scale**

Boost percentage Scale is a feature that allows Boost to scale the SMB from a user defined percentage of insulin required (up to 500%, but not recommended) at 108 mg/dl (6 mmol/l) to the user entered insulin required PCT at 180mg/dl (10 mmol/l). It can be enabled via a switch in the preferences and the percentage values are hard coded. it is only active when [Delta - Short Average Delta ] is positive, meaning that it only happens when delta variation is accelerating.

**Enhanced oref1**

If none of the above conditions are met, standard SMB logic is used to size SMBs, with the insulin required PCT entered in preferences. This only works on positive deviations and similar to the percent scale, when deltas are getting larger. Enhanced oref1 uses regular insulin sizing logic but can dose up to the Boost Bolus cap.

**Regular oref1**

Once you are outside the Boost hours, "max minutes of basal to limit SMB to for UAM" is enabled, and the dosing works in the same way as regular OpenAPSSMB.

The **BOOST** settings have a number of extra items:

Note that the default settings are designed to disable most of the functions, and you will need to adjust them.

*Boost insulin required percent* - defaults to 50% can be increased, but increasing increases hypo risk.<br>
*Boost Scale Value* - defaults to 1.0. Only increase multiplier once you have trialled. <br>
*Boost Bolus Cap* - defaults to 0.1 <br>
*UAM Boost max IOB* - defaults to 0.1 <br>
*UAM Boost Start Time (in hours using 24 hour clock)* - defaults to 7 <br>
*UAM Boost end time (in hours using 24 hour clock)* - defaults to 8
*BG level below which low glucose suspend occurs* - defaults to standard algorithm behaviour; can be adjusted between 65 and 100.<br>

**Recommended Settings**

*Boost Bolus Cap* - Start at 2.5% of TDD and increase to no more than 5% of 7 day average total daily dose. <br>
*UAM Boost max IOB* - Start at 5% of TDD and increase to no more than 15% of 7 day average total daily dose. <br>
*UAMSMBBasalMinutes* - 30 mins. This is only used overnight when IOB is large enough to trigger UAM, so it doesn't need to be a large value. <br>
*Boost insulin required percent* - recommended not to exceed 75%. Start at 50% and increase as necessary. <br>
*Target* - Set a target of 6.5mmol/l (120mg/dl) to get started with Boost. This provides a cushion as you adjust settings. Values below 5.5mmol/l (100mg/DL) are not recommended.<br>
