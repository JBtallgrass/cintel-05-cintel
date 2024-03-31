from shiny import App, render, reactive, ui
import random
from datetime import datetime, timezone

# Icons
from faicons import icon_svg

UPDATE_INTERVAL_SECS: int = 1

@reactive.calc()
def reactive_calc_combined():
    reactive.invalidate_later(UPDATE_INTERVAL_SECS)
    temp_celsius = round(random.uniform(-18, -16), 1)
    temp_fahrenheit = round((temp_celsius * 9/5) + 32, 1)
    timestamp_local = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    timestamp_gmt = datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M:%S GMT")
    return {
        "temp_celsius": temp_celsius,
        "temp_fahrenheit": temp_fahrenheit,
        "timestamp_local": timestamp_local,
        "timestamp_gmt": timestamp_gmt
    }

app_ui = ui.page_fluid(
    ui.panel_title("PyShiny Express: Live Data (Basic)"),
    ui.layout_sidebar(
        ui.panel_sidebar(
            ui.h2("Antarctic Explorer", class_="text-center"),
            ui.p("A demonstration of real-time temperature readings in Antarctica.", class_="text-center"),
        ),
        ui.panel_main(
            ui.h2("Current Temperature"),
            ui.output_text("display_temp"),
            # ui.icon("thermometer-half"), # Font Awesome icon
            ui.p("Warmer than usual"),
            ui.hr(),
            ui.h2("Current Date and Time (Local)"),
            ui.output_text("display_time_local"),
            ui.hr(),
            ui.h2("Current Date and Time (GMT)"),
            ui.output_text("display_time_gmt"),
        ),
    )
)

def server(input, output, session):
    @output
    @render.text
    def display_temp():
        latest_data = reactive_calc_combined()
        return f"{latest_data['temp_celsius']} °C / {latest_data['temp_fahrenheit']} °F"

    @output
    @render.text
    def display_time_local():
        latest_data = reactive_calc_combined()
        return latest_data['timestamp_local']

    @output
    @render.text
    def display_time_gmt():
        latest_data = reactive_calc_combined()
        return latest_data['timestamp_gmt']

app = App(app_ui, server)
