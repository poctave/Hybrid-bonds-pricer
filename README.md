###
This script allows to visualize relative value of hybrid credit bonds in terms of SSS (Sub to Senior spread) both from an historical and peers perspectives. Both systems work on Bloomberg's BQUANT (using Bloomberg data). They work by pulling the € curve of the issuer (selected in-app by user) and performing regression to interpolate over the maturity eactly equal to the Hybrid bond Next-Call-Date Rergression is pulled from bloomberg, and is done over points that match user requirements (maturity, amount outstanding) Interactive graphs are then plotted using plotly

Different model of regression (limited to the one made available by bloomberg): Nelson-Siegel, Nelson-Siegel-Svensson, Linear, Linear-Log, Log-Log. as of now Polynomial and Quadratic models are not available on BQNT/BQL even if they are enabled in NIA<GO> 

In app BQuant elements don’t allow the download of files directly in Bloomberg, this function is therefore useless for now and not called in run and the download_output is not shown. For some reason that I cant explain now, the removal of this function creates another bug where the in app element is stuck at execution of last step, this is due to Bloomberg’s BQuant usage of Voila library and this bug is currently being investigated on their side.


#Historicals:
#Arkema and Abertis appear to have no data associated when running the script on them, this bug is currently being investigated.

tickers_mapping: companies are manually imported, with their ticker and sector. Alphabetical order for companies with the trading desk scope and then alphabetical orders for others (not traded on the desk).

get_historical_data: 
	date_list = create a pandas datetimeindex with only business days from today up to (today - lookback days)
	univ_hybrid = uses Bquant (BQL) syntax to create a universe with tickers present in tickers_mapping pulling all hybrid bonds that respect the criteria (hardcoded) and returning a list of responses objects that is then transformed into a pandas dataframe: results_hybrids, dataframe contains about the hybrid bonds: Name, Issue date, Next call date (also translated in years), Ticker of Equity, mid Z spread).
	
	get_regression_request: 
		univ_senior = same as univ_hybrid but takes the whole curve (with hardcoded criteria) 
		fields_regression = empty dictionary that get filled by iterating over Next call date (years) in results_hybrids, interpolates a value at the tenor within the senior curve (using regression method defined as argument of get_historical_data).

	get_hybrid_request = packages hybrid universe and fields into one request

regression_requests: This empty list will store the BQL Request objects for retrieving the senior bond regression data for each historical date.
hybrid_requests: This empty list will store the BQL Request objects for retrieving the hybrid bond data for each historical date.
valid_dates: This empty list will store the datetime objects for the dates for which valid requests were generated. 

	date_str : loops over dates_list to convert to string with format YYY-MM-DD, creates regression requests for regressions and hybrids using functions defined before, and if it is not none, fills the empty lists with all requests. Handles error message if regression_requests is empty.

        	regression_responses = creates a list of responses from request in regression_requests that were executed all together using bq.execute_many 
        	hybrid_responses = same for hybrid_requests

all results are then zipped together in all_results (time stamp as a string with corresponding regression and hybrid data.
	SSS is only calculated if data is available and non NaN for both hybrid and senior data.

Final Try block cleans data and merges all individual df into one final: final_df 
Clean outliers using clean_z_spreads_by_change with hardcoded parameters and return final_df_clean
output_summary_table function creates a summary table with basic information, styles it using a pandas styler. Z-score values are red when low (means cheap) and green when high (means expensive). Converts and display using HTML

plot_time_series plots SSS and Zscore on a plotly graph (interactive), both components are plotted on the same column and different row as a widget (plot_output).

export_to_excel creates an output (download_output) with an HTML download link with title set as name of the ticker selected an timestamp.

# In app BQuant elements don’t allow the download of files directly in Bloomberg, this function is therefore useless for now and not called in run and the download_output is not shown. For some reason that I cant explain now, the removal of this function creates another bug where the in app element is stuck at execution of last step, this is due to Bloomberg’s BQuant usage of Voila library and this bug is currently being investigated on their side.

update_display associates values given by dropdowns to variables, then run output_summary_table and plot_time_series
# this function is not used in the version of this code, as there is no automatic refresh with observers on change in dropdown values. A simple update of the function will be necessary if we want to implement auto-update

run access current_data and bond_static_info_global, variables that are made global (still empty at this time) and populates them with data using variables given by dropdowns values and get_historical_data. A spinner widget is present during the running time of the function.

Widgets are created with minimum and maximum values

