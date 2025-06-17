#include <furi.h>
#include <gui/gui.h>
#include <input/input.h>

typedef struct {
	Gui* gui;
	ViewPort* view_port;
	FuriMessageQueue* event_queue;
} HelloWorld;

static void hello_world_draw_callback(Canvas* canvas, void* context) {
	UNUSED(context);
	canvas_clear(canvas);
	canvas_set_font(canvas, FontPrimary);
	canvas_draw_str(canvas, 30, 30, "Hello World!");
}

static void hello_world_input_callback(InputEvent* input_event, void* context) {
	HelloWorld* hello_world = context;
	furi_message_queue_put(hello_world->event_queue, input_event, FuriWaitForever);
}

int32_t hello_world_app(void* p) {
	UNUSED(p);
	
	HelloWorld* hello_world = malloc(sizeof(HelloWorld));
	hello_world->event_queue = furi_message_queue_alloc(8, sizeof(InputEvent));
	hello_world->gui = furi_record_open(RECORD_GUI);
	hello_world->view_port = view_port_alloc();
	
	view_port_draw_callback_set(hello_world->view_port, hello_world_draw_callback, hello_world);
	view_port_input_callback_set(hello_world->view_port, hello_world_input_callback, hello_world);
	
	gui_add_view_port(hello_world->gui, hello_world->view_port, GuiLayerFullscreen);
	
	InputEvent event;
	bool running = true;
	while(running) {
		if(furi_message_queue_get(hello_world->event_queue, &event, 100) == FuriStatusOk) {
			if((event.type == InputTypePress) && (event.key == InputKeyBack)) {
				running = false;
			}
		}
	}
	
	gui_remove_view_port(hello_world->gui, hello_world->view_port);
	view_port_free(hello_world->view_port);
	furi_record_close(RECORD_GUI);
	furi_message_queue_free(hello_world->event_queue);
	free(hello_world);
	
	return 0;
}
