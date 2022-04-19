# HOW to draw Something in deepStream

```sh
# move dir.
cd /opt/nvidia/deepstream/deepstream-6.0/apps/sample_apps/deepstream-app/

# rewrite file
vi deepstream_app_main.c
```



## Open deepstream_app_main.c

- 해당 파일을 수정

```sh
# find static gboolean overlay_graphics (AppCtx * appCtx, GstBuffer * buf,NvDsBatchMeta * batch_meta, guint index); FUNC.
```

- 해당 함수를 보면 text_params을 이용하여 글자를 쓰는 부분임을 알 수 있음

```c
static gboolean
overlay_graphics (AppCtx * appCtx, GstBuffer * buf,
    NvDsBatchMeta * batch_meta, guint index)
{
  
  int srcIndex = appCtx->active_source_index;

  if (srcIndex == -1)
    return TRUE;

  NvDsFrameLatencyInfo *latency_info = NULL;
  NvDsDisplayMeta *display_meta =
      nvds_acquire_display_meta_from_pool (batch_meta);

  display_meta->num_labels = 1;
  display_meta->text_params[0].display_text = g_strdup_printf ("Source: %s",
      appCtx->config.multi_source_config[srcIndex].uri);

  display_meta->text_params[0].y_offset = 20;
  display_meta->text_params[0].x_offset = 20;
  display_meta->text_params[0].font_params.font_color = (NvOSD_ColorParams) {
  0, 1, 0, 1};
  display_meta->text_params[0].font_params.font_size =
      appCtx->config.osd_config.text_size * 1.5;
  display_meta->text_params[0].font_params.font_name = "Serif";
  display_meta->text_params[0].set_bg_clr = 1;
  display_meta->text_params[0].text_bg_clr = (NvOSD_ColorParams) {
  0, 0, 0, 1.0};

  if(nvds_enable_latency_measurement) {
    g_mutex_lock (&appCtx->latency_lock);
    latency_info = &appCtx->latency_info[index];
    display_meta->num_labels++;
    display_meta->text_params[1].display_text = g_strdup_printf ("Latency: %lf",
        latency_info->latency);
    g_mutex_unlock (&appCtx->latency_lock);

    display_meta->text_params[1].y_offset = (display_meta->text_params[0].y_offset * 2 )+
      display_meta->text_params[0].font_params.font_size;
    display_meta->text_params[1].x_offset = 20;
    display_meta->text_params[1].font_params.font_color = (NvOSD_ColorParams) {
      0, 1, 0, 1};
    display_meta->text_params[1].font_params.font_size =
      appCtx->config.osd_config.text_size * 1.5;
    display_meta->text_params[1].font_params.font_name = "Arial";
    display_meta->text_params[1].set_bg_clr = 1;
    display_meta->text_params[1].text_bg_clr = (NvOSD_ColorParams) {
      0, 0, 0, 1.0};
  }

  nvds_add_display_meta_to_frame (nvds_get_nth_frame_meta (batch_meta->
          frame_meta_list, 0), display_meta);
  return TRUE;
}
```



- 이를 이용하여 line_params 등에 접근하여 원하는 그림을 그릴 수 있음

```c
void example_draw(NvDsBatchMeta * batch_meta)
{
	NvDsDisplayMeta *display_meta = nvds_acquire_display_meta_from_pool (batch_meta);
    
    float p1[2] = {0, 0};
    float p2[2] = {100, 100};

	NvOSD_LineParams *line_params = display_meta->line_params;
	line_params[display_meta->num_lines].x1 = p1[0];
    //for demonstration, user need to se these values
	line_params[display_meta->num_lines].y1 = p1[1];
    line_params[display_meta->num_lines].x2 = p2[0];
    line_params[display_meta->num_lines].y2 = p2[1];
    line_params[display_meta->num_lines].line_width = 8;
    
    line_params[display_meta->num_lines].line_color = NvOSD_ColorParams{1.0, 1.0, 1.0, 1.0};
    // color { r, g, b, A }

	display_meta->num_lines++;	//must be increase index
}
```

- 코드와 같이 NvDsBatchMeta 구조체 값을 읽어 임의의 라인을 그릴 수 있음



### NvDsDisplayMeta

[참조]: https://docs.nvidia.com/metropolis/deepstream/python-api/PYTHON_API/NvDsMeta/NvDsDisplayMeta.html

- text_params
- rect_params
- mask_rect_params
- mask_params
- line_params
- arrow_params
- circle_params

```c
struct _GstNvDsOsd
{
  /** Should be the first member when extending from GstBaseTransform. */
  GstBaseTransform parent_instance;

  /* Width of buffer. */
  gint width;
  /* Height of buffer. */
  gint height;

  /** Pointer to the nvdsosd context. */
  void *nvdsosd_context;
  /** Enum indicating how the objects are drawn,
      i.e., CPU, GPU or VIC (for Jetson only). */
  NvOSD_Mode nvdsosd_mode;

  /** Boolean value indicating whether clock is enabled. */
  gboolean show_clock;
  /** Structure containing text params for clock. */
  NvOSD_TextParams clock_text_params;

  /** List of strings to be drawn. */
  NvOSD_TextParams *text_params;
  /** List of rectangles to be drawn. */
  NvOSD_RectParams *rect_params;
  /** List of rectangles for segment masks to be drawn. */
  NvOSD_RectParams *mask_rect_params;
  /** List of segment masks to be drawn. */
  NvOSD_MaskParams *mask_params;
  /** List of lines to be drawn. */
  NvOSD_LineParams *line_params;
  /** List of arrows to be drawn. */
  NvOSD_ArrowParams *arrow_params;
  /** List of circles to be drawn. */
  NvOSD_CircleParams *circle_params;

  /** Number of rectangles to be drawn for a frame. */
  guint num_rect;
  /** Number of segment masks to be drawn for a frame. */
  guint num_segments;
  /** Number of strings to be drawn for a frame. */
  guint num_strings;
  /** Number of lines to be drawn for a frame. */
  guint num_lines;
  /** Number of arrows to be drawn for a frame. */
  guint num_arrows;
  /** Number of circles to be drawn for a frame. */
  guint num_circles;

  /** Structure containing details of rectangles to be drawn for a frame. */
  NvOSD_FrameRectParams *frame_rect_params;
  /** Structure containing details of segment masks to be drawn for a frame. */
  NvOSD_FrameSegmentMaskParams *frame_mask_params;
  /** Structure containing details of text to be overlayed for a frame. */
  NvOSD_FrameTextParams *frame_text_params;
  /** Structure containing details of lines to be drawn for a frame. */
  NvOSD_FrameLineParams *frame_line_params;
  /** Structure containing details of arrows to be drawn for a frame. */
  NvOSD_FrameArrowParams *frame_arrow_params;
  /** Structure containing details of circles to be drawn for a frame. */
  NvOSD_FrameCircleParams *frame_circle_params;

  /** Font of the text to be displayed. */
  gchar *font;
  /** Color of the clock, if enabled. */
  guint clock_color;
  /** Font size of the clock, if enabled. */
  guint clock_font_size;
  /** Border width of object. */
  guint border_width;
  /** Integer indicating the frame number. */
  guint frame_num;
  /** Boolean indicating whether text is to be drawn. */
  gboolean draw_text;
  /** Boolean indicating whether bounding is to be drawn. */
  gboolean draw_bbox;
  /** Boolean indicating whether instance mask is to be drawn. */
  gboolean draw_mask;
  /**Array containing color info for blending */
  NvOSD_Color_info color_info[MAX_BG_CLR];
  /** Boolean indicating whether hw-blend-color-attr is set. */
  gboolean hw_blend;
  /** Integer indicating number of detected classes. */
  int num_class_entries;
  /** Integer indicating gpu id to be used. */
  guint gpu_id;
  /** Pointer to the converted buffer. */
  void *conv_buf;
};
```

