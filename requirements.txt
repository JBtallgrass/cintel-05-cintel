import plotly.express as px
from shiny.express import input, ui
from shiny import render
from shinywidgets import render_plotly
import pandas as pd
import seaborn as sns
import palmerpenguins  # This package provides the Palmer Penguins dataset
from shiny import reactive
import matplotlib.pyplot as plt

# built-in function to load the Palmer Penguins dataset
penguins_df = palmerpenguins.load_penguins()

# Page name
ui.page_opts(title="JB Penguins Data", fillable=True)

color_map = {"Adelie": "blue", "Gentoo": "green", "Chinstrap": "red"}


# sidebar for user interaction
with ui.sidebar(open="open"):
    ui.h2("Sidebar")
    ui.input_selectize(
        "selected_attribute",
        "Select Plotly Attribute",
        ["bill_length_mm", "bill_depth_mm", "flipper_length_mm", "body_mass_g"],
    )

    # Create a numeric input for the number of Plotly histogram bins
    ui.input_numeric("plotly_bin_count", "Number of plotly bins", 30)

    # Creates slider input for Seaborn bins
    ui.input_slider(
        "seaborn_bin_slider",
        "Number of Bins",
        1,
        50,
        10,
    )

    # Use ui.input_checkbox_group() to create a checkbox group input to filter the species
    ui.input_checkbox_group(
        "selected_species_list",
        "Species in Scatterplot",
        ["Adelie", "Gentoo", "Chinstrap"],
        selected=["Adelie"],
        inline=True,
    )

    # Use ui.input_checkbox_group() to create a checkbox group input to filter the islands
    ui.input_checkbox_group(
        "selected_island_list",
        "Islands in Graphs",
        ["Torgersen", "Biscoe", "Dream"],
        selected=["Torgersen", "Biscoe", "Dream"],
        inline=True,
    )

# Use ui.hr() to add a horizontal rule to the sidebar
ui.hr()

# Use ui.a() to add a hyperlink to the sidebar
ui.a(
    "JBTallgrass GitHub",
    href="https://github.com/JBtallgrass/cintel-02-data",
    target="_blank",
)

# Data table showing the penguin dataset Include 2 cards with a table and a grid
with ui.layout_columns():
    with ui.card(full_screen=True):  # Full screen option
        ui.h3("Penguins Data Table")

        @render.data_frame
        def render_penguins_table():
            return render.DataTable(filtered_data())

# Create data grid
    with ui.card(full_screen=True):
        ui.h3("Penguins Data Grid")

        @render.data_frame
        def render_penguins_grid():
            return render.DataGrid(filtered_data())


# Use ui.hr() to add a horizontal rule to the sidebar
ui.hr()

# Creates a Plotly Histogram showing all species
with ui.layout_columns():
    with ui.card(full_screen=True):
        ui.h3("All Species Histogram-Plotly")

        @render_plotly
        def plotly_histogram():
            return px.histogram(
            filtered_data(),
            x="species",
            color="species",
            color_discrete_map=color_map,
            )

    with ui.card(full_screen=True):
        ui.h3("All Species ScatterPlot-plotly")

        @render_plotly
        def plotly_scatterplot():
            return px.scatter(filtered_data(),
                title="All Species ScatterPlot-plotly",
                x="body_mass_g",
                y="bill_length_mm",
                color="species",
                symbol="species",
                color_discrete_map=color_map,
            )
    # Creates a Seaborn Histogram showing all species

    with ui.card(full_screen=True):
        ui.card_header("Seaborn Histogram")

        palette = sns.color_palette("Set3")  # Choose a palette with 3 colors

        @render.plot(alt="Seaborn Histogram")
        def seaborn_histogram():
          histplot = sns.histplot(filtered_data(), x="body_mass_g", bins=input.seaborn_bin_count(), hue="species", palette=palette)
          histplot.set_title("Palmer Penguins")
          histplot.set_xlabel("Body Mass (g)")  # Set x-axis label
          histplot.set_ylabel("Count")  # Set y-axis label
          return histplot
     

    with ui.card(full_screen=True):
        ui.h3("Penguin Population by Island")

        @render_plotly()
        def island_population_chart():
            filtered = filtered_data()
            island_counts = filtered["island"].value_counts().reset_index()
            island_counts.columns = ["island", "count"]
            return px.bar(
                island_counts,
                x="island",
                y="count",
                title="Penguin Population by Island",
                labels={"count": "Number of Penguins"},
                color="island",
                color_discrete_map=color_map,
            )

    # Creates a Plotly Boxplot showing all species and islands
    with ui.card(full_screen=True):
        ui.card_header("Plotly Boxplot: Species")

        @render_plotly
        def plotly_boxplot():
            return px.box(
                filtered_data(),
                x="species",
                y=input.selected_attribute(),
                color="island",  # Add a color parameter to differentiate boxplots by island
                title="Penguins Boxplot",
                labels={
                    "species": "Species",
                    input.selected_attribute(): input.selected_attribute()
                    .replace("_", " ")
                    .title(),
                },
            )


# --------------------------------------------------------
# Reactive calculations and effects
# --------------------------------------------------------

# Add a reactive calculation to filter the data
# By decorating the function with @reactive, we can use the function to filter the data
# The function will be called whenever an input functions used to generate that output changes.
# Any output that depends on the reactive function (e.g., filtered_data()) will be updated when the data changes.


@reactive.calc
def filtered_data():
    return penguins_df[
        (penguins_df["species"].isin(input.selected_species_list()))
        & (penguins_df["island"].isin(input.selected_islands()))
    ]
