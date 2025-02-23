---
title: Correlated Logs Not Showing Up in the Trace ID Panel
kind: documentation
aliases:
  - /tracing/faq/why-cant-i-see-my-correlated-logs-in-the-trace-id-panel/
further_reading:
- link: "/tracing/other_telemetry/connect_logs_and_traces/"
  tag: "Documentation"
  text: "Correlate Traces and Logs"
- link: '/logs/guide/ease-troubleshooting-with-cross-product-correlation/'
  tag: 'Guide'
  text: 'Ease troubleshooting with cross product correlation.'
---

Clicking on a [trace][1] opens a contextual panel that contains information about the trace, about the host and the correlated logs. However the log panel can be empty in some specific cases. This page reviews how this can be fixed.

{{< img src="tracing/troubleshooting/tracing_no_logs_in_trace.png" alt="Tracing missing logs" style="width:90%;">}}

## What logs are displayed in the trace panel?

When looking at a [trace][1], there are two types of logs that can be seen:

* - `host`: Display logs from the trace's host within the trace timeframe
* - `trace_id`: Display logs that have the corresponding trace id

{{< img src="tracing/troubleshooting/tracing_logs_display_option.png" alt="Tracing log display option"  style="width:50%;">}}

## Troubleshooting steps

### Host option

If the log section is empty when the `host` option is set, go into the log explorer and check if:

- Logs are being sent from the host that emitted trace.
- There are logs for that host within the trace timeframe.
- The timestamp of the logs is properly set. Checkout [this specific guide][2] for more explanation about the log timestamp.

### Trace_id option

Make sure you have a `trace_id` standard attribute in your logs. You should see a trace icon next to the [SERVICE][3] name (black if trace is not sampled, grey if trace is sampled).

{{< img src="tracing/troubleshooting/trace_in_log_panel.png" alt="Trace icon in log panel"  style="width:50%;">}}

If your logs do not contain the `trace_id`, follow the guide on [correlating traces and logs][4].
The idea is then on the log side to:

1. Extract the trace id in a log attribute
2. Remap this attribute to the reserved `trace_id` attribute.

{{< tabs >}}
{{% tab "JSON logs" %}}

For JSON logs, step 1 and 2 are done automatically. The tracer inject the [trace][1] and [span][2] id automatically in the logs and it is remapped automatically thanks to the [reserved attribute remappers][3].

If this isn't working as expected, ensure the name of the logs attribute that contains the trace id is `dd.trace_id` and verify it is properly set in [reserved attributes][4].

{{< img src="tracing/troubleshooting/trace_id_reserved_attribute_mapping.png" alt="Trace ID mapping" >}}

[1]: /tracing/glossary/#trace
[2]: /tracing/glossary/#spans
[3]: /logs/log_configuration/processors/#remapper
[4]: https://app.datadoghq.com/logs/pipelines/remapping
{{% /tab %}}
{{% tab "With Log integration" %}}

For raw logs, using a log integration (setting the `source` attribute to: `java`, `python`, `ruby`, ...) should do all the work automatically as well.

Here is an example with the Java integration pipeline:

{{< img src="tracing/troubleshooting/tracing_java_traceid_remapping.png" alt="Java log pipeline"  style="width:90%;">}}

Now it is possible that the log format is not covered by the integration pipeline. In this case, clone the pipeline and [follow our parsing troubleshooting guide][1] to make sure it fits your format.

[1]: /logs/faq/how-to-investigate-a-log-parsing-issue/
{{% /tab %}}
{{% tab "Custom" %}}

For raw logs without any integration:

* Make sure that the custom parsing rule is extracting the [trace][1] and [span][2] IDs as a string as on the following example:

{{< img src="tracing/troubleshooting/tracing_custom_parsing.png" alt="Custom parser"  style="width:90%;">}}

* Then define a [Trace remapper][3] on the extracted attribute to remap them to the official trace id of the logs.

[1]: /tracing/glossary/#trace
[2]: /tracing/glossary/#spans
[3]: /logs/log_configuration/processors/#trace-remapper
{{% /tab %}}
{{< /tabs >}}

Once the IDs are properly injected and remapped into your logs, you can make a direct trace to log correlation:

{{< img src="tracing/troubleshooting/trace_id_injection.png" alt="Tracing id injection"  style="width:90%;">}}

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: /tracing/glossary/#trace
[2]: /logs/guide/logs-not-showing-expected-timestamp/
[3]: /tracing/glossary/#services
[4]: /tracing/other_telemetry/connect_logs_and_traces/
