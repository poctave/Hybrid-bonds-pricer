## Foreword:
This script allows to visualize relative value of hybrid credit bonds in terms of SSS (Sub to Senior spread) both from an historical and peers perspectives. Both systems work on Bloomberg's BQUANT (using Bloomberg data). They work by pulling the € curve of the issuer (selected in-app by user) and performing regression to interpolate over the maturity eactly equal to the Hybrid bond Next-Call-Date Regression is pulled from bloomberg, and is done over points that match user requirements (maturity, amount outstanding) Interactive graphs are then plotted using plotly

Different model of regression (limited to the one made available by bloomberg): Nelson-Siegel, Nelson-Siegel-Svensson, Linear, Linear-Log, Log-Log. as of now Polynomial and Quadratic models are not available on BQNT/BQL even if they are enabled in NIA<GO> 

## Note:
In app BQuant elements don’t allow the download of files directly in Bloomberg, this function is therefore useless for now and not called in run and the download_output is not shown. For some reason that I can’t explain now, the removal of this function creates another bug where the in-app element is stuck at execution of last step, this is due to Bloomberg’s BQuant usage of Voila library and this bug is currently being investigated on their side (see links).

#### BQNT Help:
https://help.bquant.blpprofessional.com/content?id=KAy6ikiv45EoSvxCAhgLeK&view=desktop#app-stuck-on-screen-displaying-executing-3-of-5 
#### GitHub:
https://github.com/voila-dashboards/voila/issues/1428 

 
# Monitor (comparables):
This script is designed to monitor hybrid bonds of sector, or a subset of companies based on several criterias, it retrieves historical data, and perform regressions to calculate Sub-to-Senior Spreads (SSS). It then handles data selection and visualisation. 
Example: distribution
<img width="1856" height="666" alt="monitor" src="https://github.com/user-attachments/assets/d3922f5a-be37-417a-bbe1-9a56c578c7e5" />

Example: Summary Table
<img width="1289" height="606" alt="table" src="https://github.com/user-attachments/assets/4c54b17b-2cde-456b-af14-8a459b7c4745" />

Example: EDF
<img width="1914" height="1024" alt="EDF" src="https://github.com/user-attachments/assets/02a96818-f381-4630-a64b-c50c93c88cc7" />

## Key Components:

#### tickers_mapping: 
companies are manually imported, with their ticker and sector. Alphabetical order for companies with the trading desk scope and then alphabetical orders for others (not traded on the desk). A list of tickers is present, it will be used later in the widgets for ticker selection by user.
#### get_historical_data(list_tickers, regression_type_choose, Min_Amt_out_choose, Max_Amt_out_choose, Min_Maturity_choose, Max_Maturity_choose): 
This function is responsible for retrieving and processing historical bond data from Bloomberg (using BQL).
##### ticker_to_sector:
unpacks tickers_mapping to map ticker directly to sector (no equity name).
##### universe:
Constructs a BQL universe of hybrid bonds using tickers from list_tickers. Filters bonds based on criteria (currency, hybrid status, amount outstanding) that are given default values (amount outstanding can be changed by user in app through widgets). Results are concatenated in results_df with sectors mapped, NANs are filled as Unknown.
##### Regression:
The idea here is to make only one BQL request (for pulling the senior unsecured curve and regressing on it) per issuer to improve computational optimization.
Constructs issuer_universe a BQL universe of sr unsecured bonds with the same criteria as for the hybrid universe + minimum value for Z-spread of >0 (to avoid bugs where bonds with false data biased the results; former 	issue found on Iberdrola).
Iterates through the current group (Equity ticker) and create several regressions requests for values in call_date_years (Next call date distance in years). Regression is pulled from Bloomberg for computational gains, 	regression is made by years over Z spread value; regression model is NSS by default but can be chosen by user through in app widgets; (models available are discussed in foreword). Request is executed only if a 		regression was actually created for the current equity group.
#### fill_dropdown(df): 
This function is responsible for populating the different widgets (dropdown and now select.multiple) present on the interface with unique values.
#### filter(df): 
This function is responsible for populating the different widgets (dropdown and select.multiple) present on the interface with unique values and outputs filtered_df. Select.multiple widgets only activates if one of the values is selected (no values  everything shows).
#### output_df(df): 
This function is responsible for taking the filtered_df and outputting it into an html table, with correct formatting (decimal places, dates format…).
#### output_summary(df): 
Creates stats on filtered_df (Count, Mean, Median, Std, Min, Max)and outputs a summary table HTML.
#### plot_sss_distribution_plotly(df): 
Takes filtered_df and creates plots (Histogram, KDE) using Plotly library. Number of bins is the minimum between 70 and the data points count /2 (or 10 if less than 10 data points). Added lines for means and [-3;+3] Z-score.
#### detach_handlers(widgets) and attach_handlers(widgets, df): 
These two functions deactivate and reactivate the listening capacity of the widgets when the refresh button is hit.
#### update_display(df, event=None): 
Uses filter, output_df, output_summary, plot_sss_distribution_plotly to perform tasks explained above when run is called (app initialization or from widgets)
#### run(event=None): 
Makes spinner visible and different tables hidden for the duration of the function takes dropdown values and assigned them as variables to be used for get_historical_data detach_handlers, update_display and re-attach_handlers. Runs export_to_excel (defined later).
#### Widgets and buttons are created: 
go_button is the refresh button, assigned to execute run on click.
#### export_to_excel: 
Exports the main results df in a timestamped excel file, creates an HTML download link

# Historicals:
This script is designed to monitor hybrid bonds of one particular company, it retrieves historical data, and perform regressions to calculate Sub-to-Senior Spreads (SSS) and Z-scores over a chosen period. It then handles data visualisation. 
Example: Veolia
<img width="1914" height="1024" alt="veolia2" src="https://github.com/user-attachments/assets/7fc780cc-f00d-4baa-b994-5ee75582ef12" />


## Key Components:

#### tickers_mapping: 
companies are manually imported, with their ticker and sector. Alphabetical order for companies with the trading desk scope and then alphabetical orders for others (not traded on the desk).
#### get_historical_data: 
##### date_list = 
create a pandas datetimeindex with only business days from today up to (today - lookback days)
##### univ_hybrid = 
Uses Bquant (BQL) syntax to create a universe with tickers present in tickers_mapping pulling all hybrid bonds that respect the criteria (hardcoded) and returning a list of responses objects that is then transformed into a pandas dataframe: results_hybrids, dataframe contains about the hybrid bonds: Name, Issue date, Next call date (also translated in years), Ticker of Equity, mid Z spread).
##### get_regression_request: 
###### univ_senior = 
same as univ_hybrid but takes the whole curve (with hardcoded criteria) 
###### fields_regression = 
Empty dictionary that get filled by iterating over Next call date (years) in results_hybrids, interpolates a value at the tenor within the senior curve (using regression method defined as argument of get_historical_data).
##### get_hybrid_request = 
Packages hybrid universe and fields into one request
##### regression_requests: 
This empty list will store the BQL Request objects for retrieving the senior bond regression data for each historical date.
##### hybrid_requests: 
This empty list will store the BQL Request objects for retrieving the hybrid bond data for each historical date.
##### valid_dates:
This empty list will store the datetime objects for the dates for which valid requests were generated. 
##### date_str:
Loops over dates_list to convert to string with format YYY-MM-DD, creates regression requests for regressions and hybrids using functions defined before, and if it is not none, fills the empty lists with all requests. Handles error message if regression_requests is empty.
##### regression_responses = 
Creates a list of responses from request in regression_requests that were executed all together using bq.execute_many 
##### hybrid_responses = 
Same for hybrid_requests

all results are then zipped together in all_results (time stamp as a string with corresponding regression and hybrid data.
SSS is only calculated if data is available and non NaN for both hybrid and senior data.

Final Try block cleans data and merges all individual df into one final: final_df 
Clean outliers using clean_z_spreads_by_change with hardcoded parameters and return final_df_clean

#### output_summary_table:
creates a summary table with basic information, styles it using a pandas styler. Z-score values are red when low (means cheap) and green when high (means expensive). Converts and display using HTML
#### plot_time_series: 
Plots SSS and Zscore on a plotly graph (interactive), both components are plotted on the same column and different row as a widget (plot_output).
#### export_to_excel:
Creates an output (download_output) with an HTML download link with title set as name of the ticker selected an timestamp.

###### In app BQuant elements don’t allow the download of files directly in Bloomberg, this function is therefore useless for now and not called in run and the download_output is not shown. For some reason that I cant explain now, the removal of this function creates another bug where the in app element is stuck at execution of last step, this is due to Bloomberg’s BQuant usage of Voila library and this bug is currently being investigated on their side.

#### update_display:
Associates values given by dropdowns to variables, then run output_summary_table and plot_time_series
###### this function is not used in the version of this code, as there is no automatic refresh with observers on change in dropdown values. A simple update of the function will be necessary if we want to implement auto-update

#### run:
Access current_data and bond_static_info_global, variables that are made global (still empty at this time) and populates them with data using variables given by dropdowns values and get_historical_data. A spinner widget is present during the running time of the function.

Widgets are created with minimum and maximum values

Run function is called, widgets are displayed with ui_display
