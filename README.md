# SAC Custom Widget Kit

Custom widgets, documentation and design guidance for SAP Analytics Cloud stories built on SAP Datasphere analytic models.

**[Open the visual guide](https://ahmedtareksdg.github.io/SAC_Custom_Widget_Kit/)**: a worked example of 51 visuals and 13 KPI card types, with their native status in SAC (QRC3 2026), when to use each, what the analytic model has to deliver, and whether to use native, configure the kit or build a new widget.

## Files

| File | What it is |
|---|---|
| [SAC_Visual_Guide.html](https://ahmedtareksdg.github.io/SAC_Custom_Widget_Kit/SAC_Visual_Guide.html) | The visual guide as a web page (filters by question, status and business area) |
| [SAC_Custom_Widgets_Complete_Guide.md](SAC_Custom_Widgets_Complete_Guide.md) | All nine chapters in one file: widget anatomy, linked analysis, Excel export, bridges, possibilities, deployment, Datasphere recipes, resources, beyond-native visuals |
| [09_Beyond_Native_Visuals_Cards_and_Features.md](09_Beyond_Native_Visuals_Cards_and_Features.md) | Chapter 09 on its own: gap matrix vs native SAC, card types, comparison guide, area playbooks, build backlog |
| [SAC_Custom_Widget_Kit.zip](SAC_Custom_Widget_Kit.zip) | The full kit: five data-bound widgets (waterfall/bridge, universal chart, Excel export table, KPI card, member selector), upload-ready builds, build tools, tests and docs |

## Use a widget

1. Unzip `SAC_Custom_Widget_Kit.zip`.
2. In SAC, go to **Stories → Custom Widgets → Create**, upload `dist/<widget>/<widget>.prod.json`, then choose `dist/<widget>/<widget>-resources.zip` under **Select File**.
3. Insert the widget in an optimized story and bind your model.

The kit's `README.md` covers building, testing and hosting.

All figures in the guide are illustrative.
