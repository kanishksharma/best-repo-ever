#--------------------------------------------------------------------#
#   TID : TID018547
#   Pre-Requisite : n/a
#   Product Area: Reporting FinancialForce - Run Report
#   Story: 	30686 - (I1.11) Create Routing
#--------------------------------------------------------------------#
describe " Validate Routing Process", :type => :request do

	before(:all) do
		gen_start_test "TID018547"
		login_user
		APEX.execute_script "ReportingBaseData.destroyData();"
		APEX.execute_script "ReportingBaseData.createReportingDefinition002();" +
							"insert ReportingHelper.newTransactionLineItemMock('0', '0', '0', '1', '001', 100.0, 200.0);" +
							"insert ReportingHelper.newTransactionLineItemMock('2', '0', '0', '1', '002', 300.0, 300.0);"	
		
		_assign_permissions  = $assignpermissiontouser + " assignPermissionSetsToUser(new List<String>{'FinancialForceReportingConfiguration', 'Reporting_Mock_Access'},'standard1');"
		APEX.execute_script _assign_permissions
	end

	after(:all) do
		gen_end_test "TID018547"
		login_user
		APEX.execute_script "ReportingBaseData.destroyData();"

		_unassign_permissions = $removepermissionsfromuser + " removePermissionsFromUser(new List<String>{'FinancialForceReportingConfiguration', 'Reporting_Mock_Access'},'standard1');"
		APEX.execute_script _unassign_permissions
	end

	it "Tests for Routing Process" do
		_expected_body = 'DIMENSION 1 DIMENSION 2 DIMENSION 3 DIMENSION 4 HOMEVALUEP1 DUALVALUEP1 HOMEVALUEP2 DUALVALUEP2 ' + 
				 'ROW 1 '+ 
				 '0 0 0 1 100.00 200.00 0.00 0.00 ' +
				 '0 100.00 200.00 0.00 0.00 ' +
				 '0 100.00 200.00 0.00 0.00 ' +
				 '0 100.00 200.00 0.00 0.00 ' +
				 '2 0 0 1 .00 0.00 300.00 400.00 ' +
				 '0 0.00 0.00 300.00 400.00 ' + 
				 '0 0.00 0.00 300.00 400.00 ' + 
				 '2 0.00 0.00 300.00 400.00'
		SF.login_as_user "std1", "S"

		test_step "TST029020 - Check that Routing process, when running report, works properly" do
			RD.run_report $ffr_reporting_definition_FFR_002
			
			# Check the url for the Report Viewer page is the correct one
			expect(page.current_url).to match (/.*\/apex\/reportingviewer\?executionId=.*&id=.*&st=Completed/)

			# Check the body is the correct one
			gen_compare _expected_body, RD.get_bodytable_text, 'The report is shown properly'
		end

		test_step "TST029071 - Check that Routing process, when clicking on reporting log, works properly" do
			SF.tab $tab_reporting_definitions
			SF.click_button_go
			RD.open_reporting_definition $ffr_reporting_definition_FFR_002
			RD.run_report_reporting_log $ffr_reporting_definition_FFR_002

			# Check the url for the Report Viewer page is the correct one
			expect(page.current_url).to match (/.*\/apex\/reportingviewer\?executionId=.*&id=.*&st=Completed/)

			# Check the body is the correct one
			gen_compare _expected_body, RD.get_bodytable_text, 'The report is shown properly'
		end	
	end
end
